.. title: Blockerbugs Infrastructure SOP
.. slug: infra-blockerbugs
.. date: 2013-11-19
.. taxonomy: Contributors/Infrastructure

==============================
Blockerbugs Infrastructure SOP
==============================

`Blockerbugs <https://pagure.io/fedora-qa/blockerbugs>`_ is an app developed by
Fedora QA to aid in tracking items related to release blocking and freeze
exception bugs in branched Fedora releases.

Contents
========

1. Contact Information
2. File Locations
3. Upgrade Process
   * Upgrade Preparation (for all upgrades)
   * Minor Upgrade (no db change)
   * Major Upgrade (with db changes)


Contact Information
===================

Owner
	Fedora QA Devel

Contact
	#fedora-qa

Location
	Phoenix

Servers
	blockerbugs01.phx2, blockerbugs02.phx2, blockerbugs01.stg.phx2

Purpose
	Hosting the `blocker bug tracking application
    <https://pagure.io/fedora-qa/blockerbugs>`_ for QA

File Locations
==============

``/etc/blockerbugs/settings.py`` - configuration for the app


Node Roles
----------

blockerbugs01.stg.phx2
  the staging instance, it is not load balanced

blockerbugs01.phx2
  one of the load balanced production nodes, it is
  responsible for running bugzilla/bodhi/koji sync

blockerbugs02.phx2
  the other load balanced production node. It does
  not do any sync operations


Building for Infra
==================

Do not use mock
---------------

For whatever reason, the ``epel7-infra`` koji tag rejects SRPMs with the
``el7.centos`` dist tag. Make sure that you build SRPMs with::

  rpmbuild -bs --define='dist .el7' blockerbugs.spec

Also note that this expects the release tarball to be in
``~/rpmbuild/SOURCES/``.

Building with Koji
------------------

You'll need to ask someone who has rights to build into ``epel7-infra`` tag to
make the build for you::

  koji build epel7-infra blockerbugs-0.4.4.11-1.el7.src.rpm

.. note:: The fun bit of this is that ``python-flask`` is only available on
  ``x86_64`` builders. If your build is routed to one of the non-x86_64, it
  will fail. The only solution available to us is to keep submitting the build
  until it's routed to one of the x86_64 builders and doesn't fail.

Once the build is complete, it should be automatically tagged into
``epel7-infra-stg`` (after a ~15 min delay), so that you can test it on
blockerbugs staging instance. Once you've verified it's working well, ask
someone with infra rights to move it to ``epel7-infra`` tag so that you can
update it in production.


Upgrading
=========

Blockerbugs is currently configured through ansible and all configuration
changes need to be done through ansible.


Upgrade Preparation (all upgrades)
----------------------------------

Blockerbugs is not packaged in epel, so the new build needs to exist in
the infrastructure stg repo for deployment to stg or the infrastructure
repo for deployments to production.

See the blockerbugs documentation for instructions on building a
blockerbugs RPM.


Minor Upgrades (no database changes)
------------------------------------

Run the following on **both** ``blockerbugs01.phx2`` and ``blockerbugs02.phx2``
if updating in production.

1. Update ansible with config changes, push changes to the ansible repo::

    roles/blockerbugs/templates/blockerbugs-settings.py.j2

2. Clear yum cache and update the blockerbugs RPM::

    yum clean expire-cache && yum update blockerbugs

3. Restart httpd to reload the application::

    service httpd restart


Major Upgrades (with database changes)
--------------------------------------
Run the following on **both** ``blockerbugs01.phx2`` and ``blockerbugs02.phx2``
if updating in production.

1. Update ansible with config changes, push changes to the ansible repo::

    roles/blockerbugs/templates/blockerbugs-settings.py.j2

2. Stop httpd on **all** relevant instances (if load balanced)::

    service httpd stop

3. Clear yum cache and update the blockerbugs RPM on all relevant instances::

    yum clean expire-cache && yum update blockerbugs

5. Upgrade the database schema::

    blockerbugs upgrade_db

6. Check the upgrade by running a manual sync to make sure that nothing
   unexpected went wrong::

    blockerbugs sync

7. Start httpd back up::

    service httpd start
