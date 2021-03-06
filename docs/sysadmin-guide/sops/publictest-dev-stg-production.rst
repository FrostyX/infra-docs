.. title: Infrastucture Machine Classifications
.. slug: infra-machine-classes
.. date: 2011-10-30
.. taxonomy: Contributors/Infrastructure

=====================================
Fedora Infrastructure Machine Classes
=====================================

Contact Information
===================

Owner
	sysadmin-main, application developers
Contact
	sysadmin-main
Location
	Everywhere we have machines.
Servers
	publictest, dev, staging, production
Purpose
	Explain our use of various types of machines.

Introduction
============

This document explains what are various types of machines are used for in
the life cycle of providing an application or resource.

Public Test machines
====================

publictest instances are used for early investigation into a resource or application.
At this stage the application might not be packaged yet, and we want to see if it's
worth packaging and starting it on the process to be available in production.
These machines are accessable to anyone in the sysadmin-test group, and coordination
of use of instances is done on an ad-hock basis. These machines are re-installed
every cycle cleanly, so all work must be saved before this occurs.

Authentication must not be against the production fas server.  We have
fakefas.fedoraproject.org setup for these systems instead.

.. note:: We're planning on merging publictest into the development servers.
    Environment-wise they'll be mostly the same (one service per machine, a
    group to manage them, no proxy interaction, etc) Service by service we'll
    assign timeframes to the machines before being rebuilt, decommissioned if
    no progress, etc.

Development
===========

These instances are for applications that are packaged and being investigated for
deployment. Typically packages and config files are modified locally to get the
application or resource working. No caching or proxies are used. Access is to a
specific sysadmin group for that application or resource. These instances can
be re-installed on request to 'start over' getting configration ready.

Some services hosted on dev systems are for testing new programs.  These will
usually be associated with an RFR and have a limited lifetime before the new
service has to prove itself worthy of continued testing, to be moved on to
stg, or have the machine decommissioned.  Other services are for developing
existing services.  They are handy if the setup of the service is tricky or
lengthy and the person in charge wants to maintain the .dev server so that
newer contributors don't have to perform that setup in order to work on the
service.

Authentication must not be against the production fas server.  We have
fakefas.fedoraproject.org setup for these systems instead.

.. note:: fakefas will be renamed fas01.dev at some point in the future

Staging
=======

These instances are used to integrate the application or resource into ansible
as well as proxy and caching setups. These instances should use ansible to deploy
all parts of the application or resource possible. Access to these instances
is only to a sysadmin group for that application, who may or may not have sudo
access.  Permissions on stg mirror permissions on production (for instance,
sysadmin-web would have access to the app servers in stg the same as
production).

Production
==========

These instances are used to serve the ready for deployment application to the public.
All changes are done via ansible and access is restricted. Changes should be done
here only after testing in staging.
