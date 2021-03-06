.. title: BladeCenter Access SOP
.. slug: infra-bladecenter
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

=====================================
BladeCenter Access Infrastructure SOP
=====================================

Many of the builders in PHX are blades in a blade center.
A few other machines are also on blades. 

Contents
========

1. Contact Information
2. Common Tasks

  1. Logging into the web interface
  2. Using the Serial Console of Blades

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Location
	 PHX
Purpose
	 Contains blades used for buildsystems, etc

Common Tasks
============

Logging into the web interface
------------------------------

The web interface to the bladecenters let you reset power, etc. 
They are bc01-mgmt and bc02-mgmt. 

Using the Serial Console of Blades
----------------------------------

All of the blades are set up with a serial console over lan (SOL). To use
this, ssh into the bladecenter. You can then pick your system and bring up
a console with::

  env -T system:blade[x]
  console -o

where x is the blade number (can be determined from web interface, etc)

To leave the console session, press Esc (

For more details on BladeCenter SOL, see
http://www-304.ibm.com/systems/support/supportsite.wss/docdisplay?brandind=5000008&lndocid=MIGR-54666


