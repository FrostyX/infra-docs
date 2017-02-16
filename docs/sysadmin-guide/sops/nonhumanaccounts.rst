.. title: Non-human Accounts Infrastructure SOP
.. slug: infra-nonhuman-accounts
.. date: 2015-03-23
.. taxonomy: Contributors/Infrastructure

=====================================
Non-human Accounts Infrastructure SOP
=====================================

We have many non-human accounts for various services, used by our web
applications and certain automated scripts.

Contents
========

1. Contact Information
2. FAS Accounts
3. Bugzilla Accounts
4. PackageDB Owners
5. Koji Accounts

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin
Persons: 
  sysadmin-main
Purpose: 
  Provide Non-human accounts to our various services

FAS Accounts
============

A FAS account should be created when a script or application needs...

* to query FAS information
* filesystem privileges associated with a group in FAS
* bugzilla privileges associated with the "fedorabugs" group.

Be sure to check if Infrastructure already has a general-purpose account
that can be used before creating a new one.

Creating a FAS account
----------------------

1. Go through the normal user creation process at
    [50]https://admin.fedoraproject.org/accounts/

    1. Set the name to: (naming convention here)
    2. Set the email to the contact email for the account (this may need
        to be done manually if the contact email is an @fedoraproject.org
        address)

2. Have a FAS admin set the account status to "bot" and set its UID below
    10000. Make sure to check that this does not break any group
    references or file ownerships first.

    * On db-fas01, using ``$ sudo -u postgres psql fas2``

      - Set it to a bot account so its not inactivated::

            => UPDATE people SET status='bot' WHERE username='username';

      - delete references to the current uid::

            => delete from visit_identity where user_id in (select id from
            people where username = 'username');

      - Find the last used id in the range we use for bots::

            => select id, username from people where id < 10000 order by id;

      - Set the account to use the newid  This should be one more than
        the largest id returned by the previous query::

            => UPDATE people SET id=NEWID WHERE username='username';

3. Get the account into any necessary groups for permissions that it may
   need. Common ones include:
   
  * Wiki editing: cla_done
  * Access to SSH keys for third party users: thirdparty
  * Access to SSH keys and password hashes for _internal_ fasClient
    runs: fas-systems

4. Document this account at:
   https://fedoraproject.org/wiki/PackageDB_admin_requests#Pseudo-users_and_Groups_for_SIGs


Alternative
-----------

This can also be achieve using SQL statements directly:

    - Find the last used id in the range we use for bots::

        => select id, username from people where id < 10000 order by id;

    - Insert the new user::

        => insert into people (id,username,human_name,password,email,status)
            values (id, 'name','small description', 'something',
                    'contact email', 'bot');

    - Find your own user id::

        => select id, username from people where username='your username';

    - Find the id of the most used groups::

        => select id, name from groups where name
            in ('cla_done', 'packager', 'fedorabugs');

    - Add the groups required::

        => insert into person_roles(person_id, group_id, role_type, sponsor_id)
           values (new_user_id, group_id, 'user', your_own_user_id);

The final steps remains the same though: document this account at:
    https://fedoraproject.org/wiki/PackageDB_admin_requests#Pseudo-users_and_Groups_for_SIGs


Bugzilla Accounts
=================

A Bugzilla account should be created when a script or application needs...

* to query or file Fedora bugs automatically

Please make sure to coordinate with the QA and Bug Triaging teams if the
script or application involves making mass changes to bugs.

If a bugzilla account needs "fedorabugs" permissions, follow the above
steps for a FAS Account first, then follow these instructions with the
email address you entered above. If the bugzilla account will not need
"fedorabugs" permissions but will still require an @fedoraproject.org
email, create an alias for that account first.

1. Create a bugzilla account as normal at
    [51]https://bugzilla.redhat.com/, using proper contact email for the
    account.
2. Document this account at (insert location here)

PackageDB Owners
================

Tie together FAS account and Bugzilla account info here

Koji Accounts
=============

TODO
