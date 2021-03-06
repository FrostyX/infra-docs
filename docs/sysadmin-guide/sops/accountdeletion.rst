.. title: Account Deletion SOP
.. slug: infra-fas-account-deletion
.. date: 2013-05-08
.. taxonomy: Contributors/Infrastructure

====================
Account Deletion SOP
====================

For the most part we do not delete accounts. In the case that a deletion
is paramount, it will need to be coordinated with appropriate entities.

Disabling accounts is another story but is limited to those with the
appropriate privileges. Reasons for accounts to be disabled can be one of
the following:

     * Person has placed SPAM on the wiki or other sites.
     * It is seen that the account has been compromised by a third party.
     * A person wishes to leave the Fedora Project and wants the account
       disabled.

Contents
--------

* Disabling

  - Disable Accounts
  - 1.2 Disable Groups

* User Requested disables

* Renames

  - Rename Accounts
  - Rename Groups

* Deletion

  - Delete Accounts
  - Delete Groups


Disable
=======

Disabling accounts is the easiest to accomplish as it just blocks people
from using their account. It does not remove the account name and
associated UID so we don't have to worry about future, unintentional
collisions.

Disable Accounts
----------------

To begin with, accounts should not be disabled until there is a ticket in
the Infrastructure ticketing system. After that the contents inside the
ticket need to be verified (to make sure people aren't playing pranks or
someone is in a crappy mood). This needs to be logged in the ticket (who
looked, what they saw, etc). Then the account can be disabled.::

  ssh db02
  sudo -u postgres pqsql fas2

  fas2=# begin;
  fas2=# select * from people where username = 'FOOO';


Here you need to verify that the account looks right, that there is only
one match, or other issues. If there are multiple matches you need to
contact one of the main sysadmin-db's on how to proceed.::

  fas2=# update people set status = 'admin_disabled' where username = 'FOOO';
  fas2=# commit;
  fas2=# /q

Disable Groups
--------------

There is no explicit way to disable groups in FAS2. Instead, we close the
group for adding new members and optionally remove existing members from
it. This can be done from the web UI if you are an administrator of the
group or you are in the accounts group. First, go to the group info page.
Then click the (edit) link next to Group Details. Make sure that the
Invite Only box is checked. This will prevent other users from requesting
the group on their own.

If you want to remove the existing users, View the Group info, then click
on the View Member List link. Click on All under the Results heading. Then
go through and click on Remove for each member.

Doing this in the database instead can be quicker if you have a lot of
people to remove. Once again, this requires someone in sysadmin-db to do
the work::

  ssh db02
  sudo -u postgres pqsql fas2

  fas2=# begin;
  fas2=# update group, set invite_only = true where name = 'FOOO';
  fas2=# commit;
  fas2=# begin;
  fas2=# select p.name, g.name, r.role_status from people as p, person_roles as r, groups as g
  where p.id = r.person_id and g.id = r.group_id
  and g.name = 'FOOO';
  fas2=# -- Make sure that the list of users in the groups looks correct
  fas2=# delete from person_roles where person_roles.group_id = (select id from groups where g.name = 'FOOO');
  fas2=# -- number of rows in both of the above should match
  fas2=# commit;
  fas2=# /q

User Requested Disables
=======================

According to our Privacy Policy, a user may request that their personal
information from FAS if they want to disable their account.  We can do this
but need to do some extra work over simply setting the account status to
disabled.

Record User's CLA information
-----------------------------

If the user has signed the CLA/FPCA, then they may have contributed something
to Fedora that we'll need to contact them about at a later date.  For that, we
need to keep at least the following information:

* Fedora username
* human name
* email address

All of this information should be on the CLA email that is sent out when a
user signs up.  We need to verify with spot (Tom Callaway) that he has that
record.  If not, we need to get it to him.  Something like::

    select id, username, human_name, email, telephone, facsimile, postal_address from people where username = 'USERNAME';

and send it to spot to keep.

Remove the personal information
-------------------------------

The following sequence of db commands should do it::

  fas2=# begin;
  fas2=# select * from people where username = 'USERNAME';

Here you need to verify that the account looks right, that there is only
one match, or other issues. If there are multiple matches you need to
contact one of the main sysadmin-db's on how to proceed.::

  fas2=# update people set human_name = '', gpg_keyid = null, ssh_key = null, unverified_email = null, comments = null, postal_address = null, telephone = null, facsimile = null, affiliation = null, ircnick = null, status = 'inactive', locale = 'C', timezone = null, latitude = null, longitude = null, country_code = null, email = 'disabled1@fedoraproject.org'  where username = 'USERNAME';

Make sure only one record was updated::

  fas2=# select * from people where username = 'USERNAME';

Make sure the correct record was updated::

  fas2=# commit;

.. note:: The email address is both not null and unique in the database.  Due
    to this, you need to set it to a new string for every user who requests
    deletion like this.

Renames
=======
In general, renames do not require as much work as deletions but they
still require coordination. This is because renames do not change the
UID/GID but some of our applications save information based on
username/groupname rather than UID/GID.

Rename Accounts
---------------

.. warning:: Needs more eyes
   This list may not be complete.

* Check the databases for koji, pkgdb, and bodhi for occurrences of the
  old username and update them to the new username.
* Check fedorapeople.org for home directories and yum repositories under
  the old username that would need to be renamed
* Check (or ask the user to check and update) mailing list subscriptions
  on fedorahosted.org and lists.fedoraproject.org under the old
  username@fedoraproject.org email alias
* Check whether the user has a username@fedoraproject.org bugzilla
  account in python-fedora and update that. Also ask the user to update
  that in bugzilla.
* If the user is in a sysadmin-* group, check for home directories on
  bastion and other infrastructure boxes that are owned by them and
  need to be renamed (Could also just tell the user to backup any files
  there themselves b/c they're getting a new home directory).
* grep through ansible for occurrences of the username
* Check for entries in trac on fedorahosted.org for the username as an
  "Assigned to" or "CC" entry.
* Add other places to check here

Rename Groups
-------------

.. warning:: Needs more eyes
   This list may not be complete.

* grep through ansible for occurrences of the group name.
* Check for group-members,group-admins,group-sponsors@fedoraproject.org
  email alias presence in any fedorahosted.org or
  lists.fedoraproject.org mailing list
* Check for entries in trac on fedorahosted.org for the username as an
  "Assigned to" or "CC" entry.
* Add other places to check here

Deletion
========

Deletion is the toughest one to audit because it requires that we look
through our systems looking for the UID and GID in addition to looking for
the username and password. The UID and GID are used on things like
filesystem permissions so we have to look there as well. Not catching
these places may lead to security issus should the UID/GID ever be reused.

.. note:: Recommended to rename instead
   When not strictly necessary to purge all traces of an account, it's
   highlyrecommended to rename the user or group to something like
   DELETED_oldusername instead of deleting. This avoids the problems and
   additional checking that we have to do below.

Delete Accounts
---------------

.. warning:: Needs more eyes
   This list may be incomplete. Needs more people to look at this and find
   places that may need to be updated

* Check everything for the #Rename Accounts case.
* Figure out what boxes a user may have had access to in the past. This
  means you need to look at all the groups a user may ever have been
  approved for (even if they are not approved for those groups now). For
  instance, any git*, svn*, bzr*, hg* groups would have granted access
  to hosted03 and hosted04. packager would have granted access to
  pkgs.fedoraproject.org. Pretty much any group grants access to
  fedorapeople.org.
* For those boxes, run a find over the files there to see if the UID
  owns any files on the system::

    # find / -uid 100068 -print

  Any files owned by that uid must be reassigned to another user or
       removed.

.. warning::  What to do about backups?
   Backups pose a special problem as they may contain the uid that's being
   removed. Need to decide how to handle this

* Add other places to check here

Delete Groups
-------------

.. warning:: Needs more eyes
   This list may be incomplete. Needs more people to look at this and find
   places that may need to be updated

* Check everything for the #Rename Groups case.
* Figure out what boxes may have had files owned by that group. This
  means that you'd need to look at the users in that group, what boxes
  they have shell accounts on, and then look at those boxes. groups used
  for hosted would also need to add hosted03 and hosted04 to that list
  and the box that serves the hosted mailing lists.
* For those boxes, run a find over the files there to see if the GID
  owns any files on the system::

    # find / -gid 100068 -print

  Any files owned by that GID must be reassigned to another group or
  removed.

.. warning:: What to do about backups?
   Backups pose a special problem as they may contain the gid that's being
   removed. Need to decide how to handle this

* Add other places to check here
