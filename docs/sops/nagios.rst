.. title: Infrastructure Nagios SOP
.. slug: infra-nagios
.. date: 2012-07-09
.. taxonomy: Contributors/Infrastructure

============================
Fedora Infrastructure Nagios
============================

Contact Information
===================

Owner
	 sysadmin-main, sysadmin-noc
Contact
	 #fedora-admin, #fedora-noc
Location
	 Anywhere
Servers
	 noc01, noc02, noc01.stg, batcave01
Purpose
	 This SOP is to describe nagios configurations

Configuration
=============

Fedora Project runs two nagios instances, nagios (noc01)
https://admin.fedoraproject.org/nagios and nagios-external (noc02)
http://admin.fedoraproject.org/nagios-external, you must be in
the 'sysadmin' group to access them.

Apart from the two production instances, we are currently running a staging
instance for testing-purposes available through SSH at noc01.stg.

nagios (noc01)
  The nagios configuration on noc01 should only monitor general host statistics
  ansible status, uptime, apache status (up/down), SSH etc.

  The configurations are found in nagios ansible module: ansible/roles/nagios

nagios-external (noc02)
  The nagios configuration on noc02 is located outside of our main datacenter
  and should monitor our user websites/applications (fedoraproject.org, FAS,
  PackageDB, Bodhi/Updates).

  The configurations are found in nagios ansible role: roles/nagios


.. note::
  Production and staging instances through SSH:
  Please make sure you are into 'sysadmin' and 'sysadmin-noc' FAS groups
  before trying to access these hosts.

  See SSH Access SOP

NRPE
----

We are currently using NRPE to execute remote Nagios plugins on any host of
our network.

A great guide about it and its usage mixed up with some nice images about
its structure can be found at:
http://nagios.sourceforge.net/docs/nrpe/NRPE.pdf

Understanding the Messages
==========================

General:
--------

Nagios notifications are generally easy to read, and follow this consistent
format::

  ** PROBLEM/ACKNOWLEDGEMENT/RECOVERY alert - hostname/Check is WARNING/CRITICAL/OK **
  ** HOST DOWN/UP alert - hostname **

Reading the message will provide extra information on what is wrong.

Disk Space Warning/Critical:
----------------------------

Disk space warnings normally include the following information::

  DISK WARNING/CRITICAL/OK - free space: mountpoint freespace(MB) (freespace(%) inode=freeinodes(%)):

A message stating "(1% inode=99%)" means that the diskspace is critical not
the inode usage and is a sign that more diskspace is required.

Further Reading
---------------

* Ansible SOP
* Outages SOP
