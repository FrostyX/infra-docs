.. title: Departing Admin SOP
.. slug: infra-departing-admin
.. date: 2013-07-15
.. taxonomy: Contributors/Infrastructure

===================
Departing admin SOP
===================

From time to time admins depart the project, this SOP checks any access they may no longer need.

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Location
	 Everywhere
Servers
	 all

Description
===========

From time to time people with admin access to various parts of the project may 
Leave the project or no longer wish to contribute. This SOP attempts to list 
The process for removing access they no longer need. 

0. First, make sure that this SOP is needed. Verify the person has left the project
   and what areas they might wish to still contibute to.

1. Gather info: fas username, email address, knowledge of passwords.

2. Check the following areas with the following commands:

    email address in puppet/ansible
      - Check: ``git grep email@address``
      - Remove: ``git commit``

    koji admin
      - Check: ``koji list-permissions --user=username``
      - Remove: ``koji revoke-permission permissionname username``

    wiki pages
      - Check: look for https://fedoraproject.org/wiki/User:Username
      - Remove: delete page, or modify with info they are no longer contributing. 

    packages
      - Check: Download https://admin.fedoraproject.org/pkgdb/lists/bugzilla?tg_format=plain and grep
      - Remove: remove from cc, orphan packages or reassign. 

    fas account
      - Check: check username in fas
      - Remove: set user inactive

      .. note:: If there are scripts or files needed, save homedir of user. 

    passwords
      - Check: if departing admin knew sensitive passwords. 
      - Remove: Change passwords. 
      
      .. note:: root pw, management interfaces, etc

