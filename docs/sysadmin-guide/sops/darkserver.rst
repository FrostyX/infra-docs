.. title: Darkserver SOP
.. slug: infra-darkserver
.. date: 2012-03-22
.. taxonomy: Contributors/Infrastructure

==============
Darkserver SOP
==============

To setup a http://darkserver.fedoraproject.org based on Darkserver project
to provide GNU_BUILD_ID information for packages. A devel instance can be
seen at http://darkserver01.dev.fedoraproject.org and staging instance is
http://darkserver01.stg.phx2.fedoraproject.org/.

This page describes how to set up the server.

Contents
========

1.  Contact Information
2.  Installing the server
3.  Setting up the database
4.  SELinux Configuration
5.  Koji plugin setup
6.  Debugging


Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin
Persons: 
  kushal mether
Sponsor: 
  nirik
Location: 
  phx2
Servers: 
  darkserver01 , darkserver01.stg, darkserver01.dev
Purpose: 
  To host Darkserver


Installing the Server
=====================
::

  root@localhost# yum install darkserver


Setting up the database
=======================
We are using MySQL as database. We will need two users, one for
koji-plugin and one for darkserver.::

  root@localhost# mysql -u root
  mysql> CREATE DATABASE darkserver;
  mysql> GRANT INSERT ON darkserver.* TO kojiplugin@'koji-hub-ip' IDENTIFIED BY 'XXX';
  mysql> GRANT SELECT ON darkserver.* TO dark@'darkserver-ip' IDENTIFIED BY 'XXX';  

Setup this db configuration in the conf file under ``/etc/darkserver/darkserverweb.conf``::

  [darkserverweb]
  host=db host name
  user=dark
  password=XXX
  database=darkserver

Now setup the db tables if it is a new install.

(For this you may need to ``'GRANT * ON darkserver.*'`` to the web user, and
then ``'REVOKE * ON darkserver.*'`` after running.)

::

  root@localhost# python /usr/lib/python2.6/site-packages/darkserverweb/manage.py syncdb

SELinux Configuration
=====================

Do the follow to allow the webserver to connect to the database.::

  root@localhost# setsebool -P httpd_can_network_connect_db 1

Setting up the Koji plugin
==========================

Install the package.::

  root@localhost# yum install darkserver-kojiplugin

Then fill up the configuration file under ``/etc/koji-hub/plugins/darkserver.conf``::

  [darkserver]
  host=db host name
  user=kojiplugin
  password=XXX
  database=darkserver
  port=3306

Then enable the plugin in the koji hub configuration.

Debugging
=========
Set DEBUG to True in ``/etc/darkserver/settings.py`` file and restart Apache.

