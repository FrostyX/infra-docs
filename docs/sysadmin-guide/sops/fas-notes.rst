.. title: Fedora Account System SOP
.. slug: infra-fas
.. date: 2013-04-04
.. taxonomy: Contributors/Infrastructure

=====================
Fedora Account System
=====================

Notes about FAS and how to do things in it:

- where are certs for fas accounts for koji, etc?
  on fas01 /var/lib/fedora-ca - makefile targets allow you to do
  things with them.
 
look in index.txt for certs. One's marked with an 'R' in the left-most
column are 'REVOKED'

to revoke a cert::

  cd /var/lib/fedora-ca
    
find the cert number in index.txt - the number is the 3rd column in the
file - you can match it to the user by searching for their username. You
want the highest number cert for their account.
     
once you have the number you would run (as root or fas)::

  make revoke cert=newcerts/$that_number.pem

How to gather information about a user
======================================

You'll want to have direct access to query the database for this.  The common
way is to have someone in sysadmin-db ssh to the postgres db hosting FAS
(currently db01).  Then access it via ident auth on the box::

    sudo -u postgres psql fas2


There are several tables that will have information about a user.  Some of it
is redundant but it's good to check all the sources there shouldn't be
inconsistencies::

    select * from people where username = 'USERNAME';

Of interest here are:

:id: for later queries
:password_changed: tells when the password was last changed
:last_seen: last login to fas (including through jsonfas from other TG1/2
    apps.  Maybe wiki and insight as well.  Not fedorahosted trac, shell
    login, etc)
:status_change: last time that the user's status was updated via the website.
    Usually triggered when the user was marked inactive for a mass password
    change and then they reset their password.

Next table is the log table::

    select * from log where author_id = ID_FROM_PREV_QUERY or description ~ '.*USERNAME.*';

The FAS writes certain events to the log table.  This will get those events.
We use both the author_id field (who made the change) and the username in a
description regex search because a few changes are made to users by admins.
Fields of interest are pretty self explanatory here:

:changetime: when the log was made
:description: description of the event that's being logged

.. note:: FAS does not log every event that happens to a user.  Only
    "important" ones.  FAS also cannot record direct changes to the database
    here (for instance, when we mark accounts inactive administratively via
    the db).

Lastly, there's the groups and person_roles table.  When a user joins a group,
the person_roles table is updated to reflect the user's status in the group,
when they applied, and when they were approved::

    select groups.name, person_roles.* from person_roles, groups where person_id = ID_FROM_INITIAL_QUERY and groups.id = person_roles.group_id;

This will give you the following fields to pay attention to:

:name: Name of the group
:role_status:  If this is unapproved, it just means the user applied for it.
    If it is approved, it means they are actually in the group.
:creation: When the user applied to the group
:approval: When the user was approved to be in the group
:role_type: What role the person has or wants to have in the group
:sponsor_id: If you suspect something is suspicious with one of the roles, you
    may want to ask the sponsor if they remember sponsoring this person

Account Deletion and renaming
=============================

.. note:: see also accountdeletion.rst
    For information on how to disable, rename, and remove accounts.

Pseudo Users
============

.. note:: see also nonhumanaccounts.rst
    For information on creating pseudo user accounts for use in pkgdb/bugzilla

fas staging
===========

we have a staging fas db setup on db-fas01.stg.phx2.fedoraproject.org - it accessed
by fas01.stg.phx2.fedoraproject.org

This system is not autopopulated by production fas - it must be done manually.
To do this you must:

- dump the fas2 db on db-fas01.phx2.fedoraproject.org::

    sudo -u postgres pg_dump -C fas2 > fas2.dump
    scp fas2.dump db-fas01.stg.phx2.fedoraproject.org:/tmp

- then on fas01.stg.phx2.fedoraproject.org::

    /etc/init.d/httpd stop

- then on db02.stg.phx2.fedoraproject.org::

    echo "drop database fas2\;" | sudo -u postgres psql ; cat fas2.dump | sudo -u postgres psql

- then on fas01.stg.phx2.fedoraproject.org::

    /etc/init.d/httpd start

that should do it.
