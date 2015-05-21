.. title: Blockerbugs Infrastructure SOP 
.. slug: infra-blockerbugs 
.. date: 2013-11-19
.. taxonomy: Contributors/Infrastructure

==============================
Blockerbugs Infrastructure SOP
==============================

Blockerbugs is an app developed by Fedora QA to aid in tracking items related
to release blocking and freeze exception bugs in branched Fedora releases.

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
	blockerbugs01.phx2, blockerbugs02.phx2, blockerbugs01.stg

Purpose
	Hosting the blocker bug tracking application for QA


File Locations
==============

``/etc/blockerbugs/settings.py`` - configuration for the app


Node Roles
----------

blockerbugs01.stg 
  the staging instance, it is not load balanced

blockerbugs01.phx2
  one of the load balanced production nodes, it is
  responsible for running bugzilla/bodhi/koji sync

blockerbugs02.phx2 
  the other load balanced production node. It does
  not do any sync operations


Upgrading
=========

blockerbugs is currently configured through puppet and all configuration
changes need to be done through puppet.


Upgrade Preparation (all upgrades)
----------------------------------

blockerbugs is not packaged in epel, so the new build needs to exist in
the infrastructure-testing repo for deployment to stg or the infrastructure
for deployments to production.

See the blockerbugs documentation for instructions on building a
blockerbugs RPM.


Minor Upgrades (no database changes)
------------------------------------

Run the following on _both_ blockerbugs01.phx2 and blockerbugs02.phx2 if
updating in production.

1. Update puppet with config changes, push changes to the puppet repo::
 
    modules/blockerbugs/templates/blockerbugs-settings.py.erb

2. clear yum cache and update the blockerbugs RPM::

      yum clean expire-cache && yum update blockerbugs

3. restart httpd to reload the application::

      service httpd restart


Major Upgrades (with database changes)
--------------------------------------
Run the following on _both_ blockerbugs01.phx2 and blockerbugs02.phx2 if
updating in production.

1. Change database users in puppet to one which has access to change the
    database schema.

2. Update puppet with config changes, push changes to the puppet repo::

      modules/blockerbugs/templates/blockerbugs-settings.py.erb

3. clear yum cache and update the blockerbugs RPM on all relevant instances::

      yum clean expire-cache && yum update blockerbugs

4. stop httpd on _all_ relevant instances (if load balanced)

5. Upgrade the database schema::

      blockerbugs upgrade_db

6. Check the upgrade by running a manual sync to make sure that nothing
    unexpected went wrong.::

      blockerbugs sync

7. start httpd back up::

      service httpd restart


