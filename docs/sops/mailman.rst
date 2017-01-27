.. title: Mailman Infrastructure SOP
.. slug: infra-mailmain
.. date: 2016-10-07
.. taxonomy: Contributors/Infrastructure

==========================
Mailman Infrastructure SOP
==========================

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main, sysadmin-tools, sysadmin-hosted

Location
	phx2

Servers
	mailman01, mailman02, mailman01.stg

Purpose
	Provides mailing list services.

Description
===========

Mailing list services for Fedora projects are located on the
mailman01.phx2.fedoraproject.org server.

Common Tasks
============

Creating a new mailing list
---------------------------

* Log into mailman01
* ``sudo -u mailman mailman3 create <listname>@lists.fedora(project|hosted).org --owner <username>@fedoraproject.org --notify``

  .. note ::     
    Note that list names should make sense, and not contain the words 'fedora'
    or 'list' - the fact that it has to do with Fedora and that it's a list
    are both obvious from the domain of the email address.

  .. important:: 
    Please make sure to add a valid description to the newly
    created list. (to avoid [no description available] on listinfo index)

Removing content from archives
==============================

We don't.

It's not easy to remove content from the archives and it's generally
useless as well because the archives are often mirrored by third parties
as well as being in the INBOXs of all of the people on the mailing list at
that time. Here's an example message to send to someone who requests
removal of archived content::

   Greetings,

   We're sorry to say that we don't remove content from the mailing list archives.
   Doing so is a non-trivial amount of work and usually doesn't achieve anything
   because the content has already been disseminated to a wide audience that we do
   not control.  The emails have gone out to all of the subscribers of the mailing
   list at that time and also (for a great many of our lists) been copied by third
   parties (for instance: http://markmail.org and http://gmane.org).

   Sorry we cannot help further,

    Mailing lists and their owners

Checking Ownership
==================

Are you in need of checking who owns a certain mailing list without having
to search around on list's frontpages?

If yes, mailman have a nice tool that will help us gaining our result in a
few seconds:

Get a full list of all the mailing lists hosted on the server: (either
fedoraproject.org or fedorahosted.org)::

  sudo /usr/lib/mailman/bin/list_admins -a

See which lists are owned by example@example.com::

  sudo /usr/lib/mailman/bin/list_admins -a | grep example@example.com

Troubleshooting and Resolution
==============================

List Administration
-------------------

Specific users are marked as 'site admins' in the database. 

Please file a issue if you feel you need to have this access.

Restart Procedure
-----------------

If the server needs to be restarted mailman should come back on it's own.
Otherwise each service on it can be restarted::

  sudo service mailman3 restart
  sudo service postfix restart

How to delete a mailing list
============================

Delete a list, but keep the archives::

  sudo -u mailman mailman3 remove <listname>

