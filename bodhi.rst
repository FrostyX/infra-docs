.. title: Bodhi Infrastructure SOP
.. slug: infra-bodhi
.. date: 2016-03-03
.. taxonomy: Contributors/Infrastructure

Bodhi Infrastructure SOP

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
12. Troubleshooting and Resolution

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

    sudo -u masher bodhi-push

You can restrict the updates by release and/or request::

   sudo -u masher bodhi-push --releases f23,f22 --request stable

You can also push specific builds::

   sudo -u masher bodhi-push --builds openssl-1.0.1k-14.fc22,openssl-1.0.1k-14.fc23

This will display a list of updates that are ready to be pushed.
It also writes out the package lists to corresponding files (eg: Testing-EL6 Stable-F23)
which can be used to feed into the sigul signing tool.

Once the packages are signed you can press `y` to begin the push.


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

Run this command from the fedora-infra `ansible` directory.::

        sudo -i ansible-playbook $(pwd)/playbooks/manual/upgrade/bodhi.yml -l staging


Remove `-l staging` to upgrade production.


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
