.. title: Ipsilon Infrastucture SOP
.. slug: infra-ipsilon
.. date: 2016-03-21
.. taxonomy: Contributors/Infrastructure

=========================
Ipsilon Infrastructure SOP
=========================



Contents
========

1. Contact Information
2. Description
3. Known Issues
4. ReStarting
5. Configuration

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	 Phoenix
Servers
	ipsilon01.phx2.fedoraproject.org ipsilon02.phx2.fedoraproject.org ipsilion01.stg.phx2.fedoraproject.org. 
	 
Purpose
	Ipsilon is our central authentication service that is used to authenticate users agains FAS. It is seperate from FAS.	

Description
===========

Ipsilon is our central authentication agent that is used to authenticate users agains FAS. It is seperate from FAS. The only service that is not using this currently is the wiki. It is a web service that is presented via httpd and is load balanced by our standard haproxy setup.

Known issues
==============

No known issues at this time. There is not currently a logout option for ipsilon, but it is not considered an issue. If group memberships are updated in ipsilon the user will need to wait a few minutes for them to replicate to the all the systems.

Restarting
===============

To restart the application you simply need to ssh to the servers for the problematic region and issue an 'service httpd restart'. This should rarely be required.

Configuration
================

Configuration is handled by the ipsilon.yaml playbook in Ansible. This can also be used to reconfigure application, if that becomes nessecary.
