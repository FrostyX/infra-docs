.. title: Memcached Infrastructure SOP
.. slug: infra-memcached
.. date: 2013-06-29
.. taxonomy: Contributors/Infrastructure

============================
Memcached Infrastructure SOP
============================

Our memcached setup is currently only used for wiki sessions. With
mediawiki, sessions stored in files over NFS or in the DB are very slow.
Memcached is a non-blocking solution for our session storage.

Contents
========

1. Contact Information
2. Checking Status
3. Flushing Memcached
4. Restarting Memcached
5. Configuring Memcached

Contact Information
===================
Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main, sysadmin-web groups

Location
	PHX

Servers
	memcached03, memcached04

Purpose
	Provide caching for Fedora web applications.

Checking Status
===============

Our memcached instances are currently firewalled to only allow access from
wiki application servers. To check the status of an instance, use::

  echo stats | nc memcached0{3,4} 11211

from an allowed host.


Flushing Memcached
==================
Sometimes, wrong contents get cached, and the cache should be flushed.
To do this, use::

  echo flush_all | nc memcached0{3,4} 11211

from an allowed host.


Restarting Memcached
====================
Note that restarting an memcached instance will drop all sessions stored
on that instance. As mediawiki uses hashing to distribute sessions across
multiple instances, restarting one out of two instances will result in
about half of the total sessions being dropped.

To restart memcached::

  sudo /etc/init.d/memcached restart

Configuring Memcached
=====================
Memcached is currently setup as a role in the ansible git repo. The main
two tunables are the MAXCONN (the maximum number of concurrent
connections) and CACHESIZE (the amount memory to use for storage). These
variables can be set through $memcached_maxconn and $memcached_cachesize
in ansible. Additionally, other options (as described in the memcached
manpage) can be set via $memcached_options.
