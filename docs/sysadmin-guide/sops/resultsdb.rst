.. title: Infrastucture resultsdb SOP
.. slug: infra-resultsdb
.. date: 2014-09-24
.. taxonomy: Contributors/Infrastructure

=============
resultsdb SOP
=============

store results from taskotron tasks

Contact Information
===================

Owner
	Fedora QA Devel, Fedora Infrastructure Team
Contact
	#fedora-qa, #fedora-admin, #fedora-noc
Location
	PHX2
Servers
	resultsdb-dev01.qa, resultsdb-stg01.qa, resultsdb01.qa
Purpose
	store results from taskotron tasks

Architecture
============

ResultsDB as a system is made up of two parts - a results storage API and a
simple html based frontend for humans to view the results accessible through
that API (resultsdb and resultsdb_frontend).

Deployment
==========

The only part of resultsdb deployment that isn't currently in the ansible
playbooks is database initialization (disabled due to bug).

Once the resultsdb app has been installed, initialize the database, run::

  resultsdb init_db

Updating
========

Database schema changes are not currently supported with resultsdb and the app
can be updated like any other web application:

- update app
- restart httpd

Backup
======

All important information in ResultsDB is stored in its database - backing up
that database is sufficient for backup and restoring that database from a
snapshot is sufficient for restoring.
