.. title: Anitya Infrastructure SOP
.. slug: infra-anitya
.. date: 2016-11-30
.. taxonomy: Contributors/Infrastructure

Anitya Infrastructure SOP

Anitya is used by Fedora to track upstream project releases and maps them
to downstream distribution packages, including (but not limited to) Fedora.

Anitya production instance: https://release-monitoring.org

Anitya project page: https://github.com/fedora-infra/anitya

Contents
========

1. Contact Information
2. Building and Deploying a Release
3. Administrating release-monitoring.org


Contact Information
===================

Owner
    Fedora Infrastructure Team
Contact
    #fedora-admin, #fedora-apps
Persons
    pingou, jcline
Location
    ?
Servers
    ?
Purpose
    Map upstream releases to Fedora packages.

Releasing
=========

The first step to making a new release is creating a Git tag for the release.


Building
^^^^^^^^

After `upstream <https://github.com/fedora-infra/anitya>`_ tags a new release in Git, a new
release can be built. The specfile is stored in the `Anitya repository
<https://github.com/fedora-infra/anitya/blob/master/files/anitya.spec>`_. Refer to the
`Infrastructure repo SOP <https://infrastructure.fedoraproject.org/infra/docs/infra-repo.rst>`_
to learn how to build the RPM.


Deploying
^^^^^^^^^

Once the new version is built, it needs to be deployed. To deploy the new version, you need
`ssh access <https://infrastructure.fedoraproject.org/infra/docs/sshaccess.rst>`_ to
batcave01.phx2.fedoraproject.org and `permissions to run the Ansible playbook
<https://infrastructure.fedoraproject.org/infra/docs/ansible.rst>`_.

Once you have the proper access and permissions, run the following from
batcave01.phx2.fedoraproject.org to update the RPM::

    $ sudo -i ansible-playbook /srv/web/infra/ansible/playbooks/manual/update-packages.yml --extra-vars="target=anitya package=anitya"


# TODO migrations?


Finally, restart Apache::

    $ sudo -i ansible anitya -m shell -a "service httpd restart"

The new version should now be deployed.

Administrating release-monitoring.org
=====================================

Flags
^^^^^
Anitya lets users flag projects for administrator attention. This is accessible to administrators in the
`flags tab <https://release-monitoring.org/flags>`_.
