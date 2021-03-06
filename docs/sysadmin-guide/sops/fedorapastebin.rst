.. title: Fedora Pastebin SOP
.. slug: infra-fpaste
.. date: 2013-04-15
.. taxonomy: Contributors/Infrastructure

===================
Fedora Pastebin SOP
===================

Contents
========

1. Contact Information
2. Introduction
3. Installation
4. Dashboard
5. Add a word to censored list


1. Contact Information
-----------------------

Owner
	Fedora Infrastructure Team
Contact
	#fedora-admin
Persons
	athmane herlo
Sponsor
	nirik
Location
	phx2
Servers
	paste01.stg, paste01.dev
Purpose
	To host Fedora Pastebin


2. Introduction
----------------

Fedora pastebin is powered by sticky-notes which is included in EPEL.

Fedora theming (skin) is included in ansible role.


3. Installation
----------------

Sticky-notes needs a MySQL db and a user with 'select, update, delete, insert' privileges.

It's recommended to dump and import db from a working installation
to save time (skipping the installation and tweaking).

By default the installation is locked ie: you can't relaunch it.

However, you can unlock the installation by commenting the line containing
``$gsod->trigger`` in ``/etc/sticky-notes/install.php`` then pointing the web browser to '/install'

The configuration file containing general settings and DB credentials
is located in ``/etc/sticky-notes/config.php``

4. Dashboard
-------------

Sticky-notes has a dashboard (URL: /admin/) that can be used to :

- Manage pastes:
    - deleting paste
    - getting information about the paste author (IP/Date/time etc...)
- Manage users (aka admins) which can log into the dashboard
- Manage IP Bans (add / delete banned IPs).
- Authentication (not needed)
- Site configuration:
    - General configuration (included in config.php).
    - Project Honey Pot configuration (not a FOSS service)
    - Word censor configuration: a list of words to be censored in pastes.

5. Add a word to censored list
------------------------------

If a word is in censored list, any paste containing that word will be
rejected, to add one, edit the variable '$sg_censor' in sticky-notes configuration file.::

  $sg_censor = "WORD1
  WORD2
  ...
  ...
  WORDn";
