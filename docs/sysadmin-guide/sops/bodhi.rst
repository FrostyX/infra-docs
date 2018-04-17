.. title: Bodhi Infrastructure SOP
.. slug: infra-bodhi
.. date: 2016-03-03
.. taxonomy: Contributors/Infrastructure

========================
Bodhi Infrastructure SOP
========================

Bodhi is used by Fedora developers to submit potential package updates for
releases. From here, bodhi handles all of the dirty work, from sending
around emails, dealing with Koji, to mashing the repositories.

Bodhi production instance: https://bodhi.fedoraproject.org
Bodhi project page: https://github.com/fedora-infra/bodhi

Contents
========

1. Contact Information
2. Adding a new pending release
3. 0-day Release Actions
4. Configuring all bodhi nodes
5. Pushing updates
6. Monitoring the bodhi masher output
7. Resuming a failed push
8. Performing a production bodhi upgrade
9. Syncing the production database to staging
10. Release EOL
11. Adding notices to the front page or new update form
12. Using the Bodhi Shell to modify updates by hand
13. Troubleshooting and Resolution

Contact Information
===================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin
Persons
 lmacken
Location
 Phoenix
Servers
 bodhi-backend01
 bodhi-backend02
 bodhi03
 bodhi04
Purpose
 Push package updates, and handle new submissions.

Adding a new pending release
============================

Adding and modifying releases is done using the `bodhi-manage-releases` tool.

You can add a new pending release by running this command::

        bodhi-manage-releases create --name F23 --long-name "Fedora 23" --id-prefix FEDORA --version 23 --branch f23 --dist-tag f23 --stable-tag f23-updates --testing-tag f23-updates-testing --candidate-tag f23-updates-candidate --pending-stable-tag f23-updates-pending --pending-testing-tag f23-updates-testing-pending --override-tag f23-override --state pending                                                                                                                                                       


0-day Release Actions
=====================

- update atomic config
- run the ansible playbook

Going from pending to a proper release in bodhi requires a few steps:

Change state from pending to current::

        bodhi-manage-releases edit --name F23 --state current

You may also need to disable any pre-release policy defined in the bodhi
config in ansible.::

        ansible/roles/bodhi2/base/templates/production.ini.j2

Then you can safely remove or comment out the pre-release-specific policy.
Release status post-beta enforces the 'Beta to Pre Release' policy defined at https://fedoraproject.org/wiki/Updates_Policy

::

        f23.status = 'post_beta'
        f23.post_beta.mandatory_days_in_testing = 3
        f23.post_beta.critpath.num_admin_approvals = 1
        f23.post_beta.critpath.min_karma = 2


Configuring all bodhi nodes
===========================

Run this command from the `ansible` checkout to configure all of bodhi in production::

        sudo -i ansible-playbook $(pwd)/playbooks/groups/bodhi2.yml


Pushing updates
===============

SSH into the `bodhi-backend01` machine and run::

    $ sudo -u apache bodhi-push

You can restrict the updates by release and/or request::

    $ sudo -u apache bodhi-push --releases f23,f22 --request stable

You can also push specific builds::

    $ sudo -u apache bodhi-push --builds openssl-1.0.1k-14.fc22,openssl-1.0.1k-14.fc23

This will display a list of updates that are ready to be pushed.


Monitoring the bodhi masher output
==================================

You can monitor the bodhi masher via the systemd journal::

        journalctl -f -u fedmsg-hub


Resuming a failed push
======================

If a push fails for some reason, you can easily resume it by running::

        sudo -u masher bodhi-push --resume


Performing a bodhi upgrade
===========================

Staging
-------

Ensure that no changes are needed to the Bodhi configuration files. If they
are, make the needed changes and re-run the deployment playbooks::

        sudo rbac-playbook -l staging groups/bodhi-backend.yml
        sudo rbac-playbook -l staging groups/bodhi2.yml

Run these commands::

        sudo rbac-playbook manual/staging-sync/bodhi.yml -l staging
        sudo rbac-playbook manual/upgrade/bodhi.yml -l staging


Production
----------

Ensure that no changes are needed to the Bodhi configuration files. If they
are, make the needed changes and re-run the deployment playbooks::

        sudo rbac-playbook groups/bodhi-backend.yml -l bodhi2,bodhi-backend
        sudo rbac-playbook groups/bodhi2.yml -l bodhi2,bodhi-backend

To update the bodhi RPMs in production::

        sudo rbac-playbook manual/upgrade/bodhi.yml -l bodhi2,bodhi-backend


Syncing the production database to staging
==========================================

This can be useful for testing issues with production data in staging::

        sudo -i ansible-playbook $(pwd)/playbooks/manual/staging-sync/bodhi.yml -l staging


Release EOL
===========

::
        bodhi-manage-releases edit --name F21 --state archived


Adding notices to the front page or new update form
===================================================

You can easily add notification messages to the front page of bodhi using the `frontpage_notice` option in `ansible/roles/bodhi2/base/templates/production.ini.j2`. If you want to flash a message on the New Update Form, you can use the `newupdate_notice` variable instead. This can be useful for announcing things like service outages, etc.


Using the Bodhi Shell to modify updates by hand
===============================================

The "bodhi shell" is a Python shell with the SQLAlchemy session and transaction manager initialized.
It can be run from any production/staging backend instance and allows you to modify any models by hand.

::
        sudo pshell /etc/bodhi/production.ini

        # Execute a script that sets up the `db` and provides a `delete_update` function.
        # This will eventually be shipped in the bodhi package, but can also be found here.
        # https://raw.githubusercontent.com/fedora-infra/bodhi/develop/tools/shelldb.py
        >>> execfile('shelldb.py')

At this point you have access to a `db` SQLAlchemy Session instance, a `t`
`transaction` module, and `m` for the `bodhi.models`.


::
        # Fetch an update, and tweak it as necessary.
        >>> up = m.Update.get(u'u'FEDORA-2016-4d226a5f7e', db)

        # Commit the transaction
        >>> t.commit()


Here is an example of merging two updates together and deleting the original.

::
        >>> up = m.Update.get(u'FEDORA-2016-4d226a5f7e', db)
        >>> up.builds
        [<Build {'epoch': 0, 'nvr': u'resteasy-3.0.17-2.fc24'}>, <Build {'epoch': 0, 'nvr': u'pki-core-10.3.5-1.fc24'}>]
        >>> b = up.builds[0]
        >>> up2 = m.Update.get(u'FEDORA-2016-5f63a874ca', db)
        >>> up2.builds
        [<Build {'epoch': 0, 'nvr': u'resteasy-3.0.17-3.fc24'}>]
        >>> up.builds.remove(b)
        >>> up.builds.append(up2.builds[0])
        >>> delete_update(up2)
        >>> t.commit()


Troubleshooting and Resolution
==============================

Atomic OSTree compose failure
-----------------------------

If the Atomic OSTree compose fails with some sort of `Device or Resource busy` error, then run `mount` to see if there
are any stray `tmpfs` mounts still active::

        tmpfs on /var/lib/mock/fedora-22-updates-testing-x86_64/root/var/tmp/rpm-ostree.bylgUq type tmpfs (rw,relatime,seclabel,mode=755)

You can then `umount /var/lib/mock/fedora-22-updates-testing-x86_64/root/var/tmp/rpm-ostree.bylgUq` and resume the push again.


nfs repodata cache IOError
--------------------------

Sometimes you may hit an IOError during the updateinfo.xml generation
process from createrepo_c::

        IOError: Cannot open /mnt/koji/mash/updates/epel7-160228.1356/../epel7.repocache/repodata/repomd.xml: File /mnt/koji/mash/updates/epel7-160228.1356/../epel7.repocache/repodata/repomd.xml doesn't exists or not a regular file

This issue will be resolved with NFSv4, but in the mean time it can be worked
around by removing the `.repocache` directory and resuming the push::

        rm -fr /mnt/koji/mash/updates/epel7.repocache
