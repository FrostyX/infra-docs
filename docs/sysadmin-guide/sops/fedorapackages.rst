.. title: Fedora Packages SOP
.. slug: infra-fedora-packages
.. date: 2018-02-14
.. taxonomy: Contributors/Infrastructure

===================
Fedora Packages SOP
===================

This SOP is for the Fedora Packages web application.
https://apps.fedoraproject.org/packages

Contents
========

1. Contact Information
2. Building a new release
3. Deploying to the development server
4. Maintenance
5. Checking for AGPL violations

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, #fedora-apps

Persons
	cverna

Location
	PHX2

Servers
	packages03.phx2.fedoraproject.org
	packages04.phx2.fedoraproject.org
	packages03.stg.phx2.fedoraproject.org

Purpose
	Web interface for searching packages information

Releasing
=========
To create a new release simply create a new Git tag for the release.

Building
--------
After `upstream <https://github.com/fedora-infra/fedora-packages>`_ tags a new release in Git, a new
release can be built. The specfile is stored in the `fedora-packages repository
<https://github.com/fedora-infra/fedora-packages/blob/develop/fedora-packages.spec>`_. Refer to the
`Infrastructure repo SOP <https://docs.pagure.org/infra-docs/sysadmin-guide/sops/infra-repo.html>`_
to learn how to build the RPM.

Deploying
---------

Once the new version is built, it needs to be deployed. To deploy the new version, you need
`ssh access <https://infrastructure.fedoraproject.org/infra/docs/sshaccess.rst>`_ to
batcave01.phx2.fedoraproject.org and `permissions to run the Ansible playbook
<https://infrastructure.fedoraproject.org/infra/docs/ansible.rst>`_.

All the following commands should be run from batcave01.

Upgrading
---------

To upgrade, run the upgrade playbook::

    $ sudo rbac-playbook manual/upgrade/packages.yml

This will upgrade the fedora-pacages package and restart the Apache web server
and fedmsg-hub service.

Rebuild the xapian Database
---------------------------

If you need to rebuild the xapian database then you can run the following playbook::

	$ sudo rbac-playbook manual/rebuild/fedora-packages.yml


Maintenance
===========

The web application is served by httpd and managed by the httpd service.::

	$ sudo systemctl restart httpd

can be used to restart the service if needed.
The application log files are available under `/var/log/httpd/` directory.

The xapian database is updated by a fedmsg consumer.
You can restart the fedmsg-hub serivce if needed by using::

	$ sudo systemctl restart fedmsg-hub

To check the consumer logs you can use::

	$ sudo journalctl -u fedmsg-hub


Checking for AGPL violations
============================

To remain AGPL compliant, we must ensure that all modifications to the code
are made available in the SRPM that we link to in the footer of the
application. You can easily query our app servers to determine if any AGPL
violating code modifications have been made to the package.::

  func-command --host="*app*" --host="community*" "rpm -V fedoracommunity"

You can safely ignore any changes to non-code files in the output. If any
violations are found, the Infrastructure Team should be notified immediately.
