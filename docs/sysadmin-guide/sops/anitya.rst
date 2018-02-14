.. title: Anitya Infrastructure SOP
.. slug: infra-anitya
.. date: 2016-11-30
.. taxonomy: Contributors/Infrastructure

=========================
Anitya Infrastructure SOP
=========================

Anitya is used by Fedora to track upstream project releases and maps them
to downstream distribution packages, including (but not limited to) Fedora.

Anitya production instance: https://release-monitoring.org

Anitya project page: https://github.com/release-monitoring/anitya


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
    anitya-backend01.vpn.fedoraproject.org
    anitya-frontend01.vpn.fedoraproject.org
Purpose
    Map upstream releases to Fedora packages.

Hosts
=====
The current deployment is made up of two hosts, anitya-backend01 and
anitya-frontend01.

anitya-frontend01
-----------------
This host runs:

- The apache/mod_wsgi application for release-monitoring.org

- A fedmsg-relay instance for anitya's local fedmsg bus

This host relies on:

- A postgres db server running on anitya-backend01

- Lots of external third-party services.  The anitya webapp can scrape
  pypi, rubygems.org, sourceforge and many others on command.

Things that rely on this host:

- The Fedora Infrastructure bus subscribes to the anitya bus published
  here by the local fedmsg-relay daemon at tcp://release-monitoring.org:9940

- the-new-hotness is a fedmsg-hub plugin running in FI on hotness01.  It
  listens for anitya messages from here and performs actions on koji and
  bugzilla.

- anitya-backend01 expects to publish fedmsg messages via
  anitya-frontend01's fedmsg-relay daemon.  Access should be restricted by
  firewall.

anitya-backend01
----------------
This is responsible for running the anitya backend cronjobs. It also is
the host for the Anitya PostgreSQL database server.

The services and jobs on this host are:

- A cronjob that retrieves all projects from the PostgreSQL database and
  checks the upstream project to see if there's a new version. This is run
  every 12 hours.

- A PostgreSQL database server to be used by that cron job and by
  anitya-frontend01.

- A database backup job that runs daily. Database dumps are available at
  `the normal database dump location
  <https://infrastructure.fedoraproject.org/infra/db-dumps/>`_.

This host relies on:
- The fedmsg-relay daemon running on anitya-frontend01.

- Lots of external third-party services.  The cronjob makeall kinds of
  requests out to the Internet that can fail in various ways.

Things that rely on this host:

- The webapps running on anitya-frontend01 relies on the postgres db
  server running on this node.


Releasing
=========

The first step to making a new release is creating a Git tag for the release.

Building
--------
After `upstream <https://github.com/fedora-infra/anitya>`_ tags a new release in Git, a new
release can be built. The specfile is stored in the `Anitya repository
<https://github.com/fedora-infra/anitya/blob/master/files/anitya.spec>`_. Refer to the
`Infrastructure repo SOP <https://docs.pagure.org/infra-docs/sysadmin-guide/sops/infra-repo.html>`_
to learn how to build the RPM.

Deploying
---------
At the moment, there is no staging deployment of Anitya.

Once the new version is built, it needs to be deployed. To deploy the new version, you need
`ssh access <https://infrastructure.fedoraproject.org/infra/docs/sshaccess.rst>`_ to
batcave01.phx2.fedoraproject.org and `permissions to run the Ansible playbook
<https://infrastructure.fedoraproject.org/infra/docs/ansible.rst>`_.

All the following commands should be run from batcave01.

Configuration
^^^^^^^^^^^^^
First, ensure there are no configuration changes required for the new update. If there are,
update the Ansible anitya role(s) and optionally run the deployment playbook::

    $ sudo rbac-playbook groups/anitya.yml

The anitya roles are applied during the upgrade playbook so this is merely an opportunity to
test your configuration changes prior to a package upgrade.

Upgrading
^^^^^^^^^
Both anitya-backend and anitya-frontend need the new version of the ``anitya`` package.
To upgrade, run the upgrade playbook::

    $ sudo rbac-playbook manual/upgrade/anitya.yml

This will upgrade the anitya package on both the frontend and the backend, perform any
database migrations with Alembic, and restart the Apache web server.

Congratulations! The new version should now be deployed.


Administrating release-monitoring.org
=====================================
Anitya offers some tools to administer the web application itself. These are useful
for when users accidentally create duplicate projects, versions found get messed up,
etc.

Flags
-----
Anitya lets users flag projects for administrator attention. This is accessible to
administrators in the `flags tab <https://release-monitoring.org/flags>`_.
