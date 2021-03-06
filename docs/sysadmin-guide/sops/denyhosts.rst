.. title: Denyhosts Infrastructure SOP
.. slug: infra-denyhosts
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

============================
Denyhosts Infrastructure SOP
============================

Denyhosts provides a protection against brute force attacks.

Contents
========

1. Contact Information
2. Description
3. Troubleshooting and Resolution

  1. Connection issues

Contact Information
====================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main group

Location
	Anywhere

Servers
	All

Purpose
	Denyhosts provides a protection against brute force attacks.

Description
===========

All of our servers now implement denyhosts to protect against brute force
attacks. Very few boxes should be in the 'allowed' list. Especially
internally.

Troubleshooting and Resolution
==============================

Connection issues
-----------------
The most common issue will be legitimate logins failing. First, try to
figure out why a host ended up on the deny list (tcptraceroute, failed
login attempts, etc are all good candidates). Next do the following
directions. The below example is for a host (10.0.0.1) being banned. Login
to the box from a different host and as root do the following.::

  cd /var/lib/denyhosts
  sed -si '/10.0.0.1/d' * /etc/hosts.deny
  /etc/init.d/denyhosts restart

That should correct the problem.

