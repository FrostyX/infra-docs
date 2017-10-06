.. title: Tag2distrepo Infrastucture SOP
.. slug: infra-tag2distrepo
.. date: 2017-10-07
.. taxonomy: Contributors/Infrastructure

===============================
Tag2DistRepo Infrastructure SOP
===============================



Contents
========

1. Contact Information
2. Description
3. Configuration

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Primary upstream contact
    Patrick Uiterwijk - FAS: puiterwijk
Location
	 Phoenix
Servers
        bodhi-backend02.phx2.fedoraproject.org
	 
Purpose
        Tag2DistRepo is a Fedmsg Consumer that waits for tag operations in specific tags, and then instructs Koji to create Distro Repos.

Description
===========

Tag2DistRepo is a Fedmsg Consumer that waits for tag operations in specific tags, and then instructs Koji to create Distro Repos.

Configuration
================

Configuration is handled by the bodhi-backend.yaml playbook in Ansible. This can also be used to reconfigure application, if that becomes nessecary.
