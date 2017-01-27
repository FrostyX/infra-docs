.. title: Bugzilla Sync SOP
.. slug: infra-bugzilla-sync
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

================================
Bugzilla Sync Infrastructure SOP
================================

We do not run bugzilla.redhat.com. If bugzilla itself is down we need to
get in touch with Red Hat IT or one of the bugzilla hackers (for instance,
Dave Lawrence (dkl)) in order to fix it.

Infrastructure has some scripts that perform administrative functions on
bugzilla.redhat.com. These scripts sync information from FAS and the
Package Database into bugzilla.

Contents
========

1. Contact Information
2. Description
3. Troubleshooting and Resolution

  1. Errors while syncing bugzilla with the PackageDB

Contact Information
===================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-admin
Persons
	abadger1999
Location
	Phoenix, Denver (Tummy), Red Hat Infrastructure
Servers
	(fas1, app5) => Need to migrate these to bapp1, bugzilla.redhat.com
Purpose
	Sync Fedora information to bugzilla.redhat.com

Description
===========

At present there are two scripts that sync information from Fedora into
bugzilla.

export-bugzilla.py
------------------

``export-bugzilla.py`` is the first script. It is responsible for syncing
Fedora Accounts into bugzilla. It adds Fedora packages and bug triagers
into a bugzilla group that gives the users extra permissions within
bugzilla. This script is run off of a cron job on FAS1. The source code
resides in the FAS git repo in ``fas/scripts/export-bugzilla.*`` however the
code we run on the servers presently lives in ansible::

  roles/fas_server/files/export-bugzilla

pkgdb-sync-bugzilla
-------------------

The other script is pkgdb-sync-bugzilla. It is responsible for syncing the
package owners and cclists to bugzilla from the pkgdb. The script runs off
a cron job on app5. The source code is in the packagedb bzr repo is
``packagedb/fedora-packagedb-stable/server-scripts/pkgdb-sync-bugzilla.*``.
Just like FAS, a separate copy is presently installed from ansbile to
``/usr/local/bin/pkgdb-sync-bugzilla`` but that should change ASAP as the
present fedora-packagedb package installs ``/usr/bin/pkgdb-sync-bugzilla``.

Troubleshooting and Resolution
==============================

Errors while syncing bugzilla with the PackageDB
------------------------------------------------

One frequent problem is that people will sign up to watch a package in the
packagedb but their email address in FAS isn't a bugzilla email address.
When this happens the scripts that try to sync the packagedb information
to bugzilla encounter an error and send an email like this::

  Subject: Errors while syncing bugzilla with the PackageDB

  The following errors were encountered while updating bugzilla with information
  from the Package Database.  Please have the problems taken care of:

  ({'product': u'Fedora', 'component': u'aircrack-ng', 'initialowner': u'baz@zardoz.org',
  'initialcclist': [u'foo@bar.org', u'baz@zardoz.org']}, 504, 'The name foo@bar.org is not a
  valid username.  \n    Either you misspelled it, or the person has not\n    registered for a
  Red Hat Bugzilla account.')

When this happens we attempt to contact the person with the problematic
mail address and get them to change it. Here's a boilerplate message::

  To: foo@bar.org
  Subject: Fedora Account System Email vs Bugzilla Email

  Hello,

  You are signed up to receive bug reports against the aircrack-ng package
  in Fedora.  Unfortunately, the email address we have for you in the
  Fedora Account System is not a valid bugzilla email address.  That means
  that bugzilla won't send you mail and we're getting errors in the script
  that syncs the cclist into bugzilla.

  There's a few ways to resolve this:

  1) Create a new bugzilla account with the email foo@bar.org as
  an account at https://bugzilla.redhat.com.

  2) Change an existing account on https://bugzilla.redhat.com to use the
  foo@bar.org email address.

  3) Change your email address in https://admin.fedoraproject.org/accounts
  to use an email address that matches with an existing bugzilla email
  address.

  Please let me know what you want to do!

  Thank you,

If the user does not reply someone in the cvsadmin group needs to go into
the pkgdb and remove the user from the cclist for the package.
