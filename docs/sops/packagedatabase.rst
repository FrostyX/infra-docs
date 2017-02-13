.. title: Package Database Infrastucture SOP
.. slug: infra-packagedb
.. date: 2013-04-30
.. taxonomy: Contributors/Infrastructure

===================================
Package Database Infrastructure SOP
===================================


The PackageDB is used by Fedora developers to manage package ownership and
acls. It controls who is allowed to commit to a package and who gets
notification of changes to packages.

PackageDB project Trac: [45]https://fedorahosted.org/packagedb/

Contents
i=======

1. Contact Information
2. Troubleshooting and Resolution
3. Common Actions

	1. Adding a new Pseudo User as a package owner
	2. Renaming a package
	3. Removing a package
	4. Add a new release
	5. Update App DB for a release going final
	6. Orphaning all the packages for a user

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin

Persons
	abadger1999

Location
	Phoenix

Servers
  admin.fedoraproject.org Click on one of the [55]current haproxy
  servers to see the physical servers

Purpose
	Manage package ownership

Troubleshooting and Resolution
==============================

Common Actions
==============

Adding a new Pseudo User as a package owner
-------------------------------------------

Sometimes you want to have a mailing list own a package so that bugzilla
email is assigned to the mailing list. Doing this requires adding a new
pseudo user to the account system and assigning that person as the package
maintainer.

.. warning:: pseudo users often have a dash in their name.  We create email
        aliases via ansible that have dashes in their name in order to not
        collide with fas usernames (users cannot create usernames with dashes
        via the webui).  Make sure that any pseudo-users you create do not
        clash with existing email aliases.

In the following examples, replace ("xen", "kernel-xen-2.6") with the
packages you are assigning to the new user and 9902 to the userid you
select in step 2

* Log into fas-db01.
* Log into the db as a user that can make changes::

    $ psql -U postgres fas2
    fas2>

  * Find the current pseudo-users::

      fas2>  select id, username from people where id < 10000 order by id;
      id  |     username
      ------+------------------
      9900 | orphan
      9901 | anaconda-maint

  * Create a new account with the next available id after 9900::

      fas2> insert into people (id, username, human_name, password, email)
      values (9902, 'xen-maint', 'Xen Maintainers', '*', 'xen-maint@redhat.com');

* Connect to the pkgdb as a user that can make changes::

   $ psql -U pkgdbadmin -h db01 pkgdb
    pkgdb>

* Add the current package owner as a comaintainer of the package. If
  this user is not currently on he acls for the package you can use the
  following database queries::

    insert into personpackagelisting (username, packagelistingid)
    select pl.owner, pl.id from packagelisting as pl, package as p
    where p.id = pl.packageid and p.name in ('xen', 'kernel-xen-2.6');
    insert into personpackagelistingacl (personpackagelistingid, acl, statuscode)
    select ppl.id, 'build', 3 from personpackagelisting as ppl, packagelisting as pl, package as p
    where p.id = pl.packageid and pl.id = ppl.packagelistingid and pl.owner = ppl.username
    and p.name in ('xen', 'kernel-xen-2.6');
    insert into personpackagelistingacl (personpackagelistingid, acl, statuscode)
    select ppl.id, 'commit', 3 from personpackagelisting as ppl, packagelisting as pl, package as p
    where p.id = pl.packageid and pl.id = ppl.packagelistingid
    and pl.owner = ppl.username
    and p.name in ('xen', 'kernel-xen-2.6');
    insert into personpackagelistingacl (personpackagelistingid, acl, statuscode)
    select ppl.id, 'approveacls', 3 from personpackagelisting as ppl, packagelisting as pl, package as p
    where p.id = pl.packageid and pl.id = ppl.packagelistingid
    and pl.owner = ppl.username
    and p.name in ('xen', 'kernel-xen-2.6');


  If the owner is in the acls, you will need to figure out which packages
  already acls and only add the new acls for that one.

* Reassign the pseudo-user to be the new owner::

    update packagelisting set owner = 'xen-maint' from package as p
    where packagelisting.packageid = p.id and p.name in ('xen', 'kernel-xen-2.6');

Renaming a package
-------------------

On db2::

 sudo -u postgres psql pkgdb
 select * from package where name = 'OLDNAME';
 [Make sure only the package you want is selected]
 update package set name = 'NEWNAME' where name = 'OLDNAME';

On cvs-int::

 CVSROOT=/cvs/pkgs cvs co CVSROOT
 sed -i 's/OLDNAME/NEWNAME/g' CVSROOT/modules
 cvs commit -m 'Rename OLDNAME => NEWNAME'
 cd /cvs/pkgs/rpms
 mv OLDNAME NEWNAME
 cd NEWNAME
 find . -name 'Makefile,v' -exec sed -i 's/NAME := OLDNAME/NAME := NEWNAME/' \{\} \;
 cd ../../devel
 rm OLDNAME
 ln -s ../rpms/NEWNAME/devel .

If the package has existed long enough to have been added to koji, run
something like the following to "retire" the old name in koji.::

 koji block-pkg dist-f12 OLDNAME

Removing a package
==================

.. warning::
  Do not remove a package if it has been built for a fedora release or if
  you are not also willing to remove the cvs directory.

When a package has been added due to a typo, it can be removed in one of
two ways: marking it as a mistake with the "removed" status or deleting it
from the db entirely. Marking it as removed is easier and is explained
below.

On db2::

  sudo -u postgres psql pkgdb
  pkgdb=# select id, name, summary, statuscode from package where name = 'b';
    id  | name |                     summary                      | statuscode
  ------+------+--------------------------------------------------+-----------
   6618 | b    | A simple database interface to MS-SQL for Python |          3
  (rows 1)

- Make sure there is only one package returned and it is the correct one.
- Statuscode 3 is "approved" and it's what we're changing from
- You'll also need the id for later::

    pkgdb=# BEGIN;
    pkgdb=# update package set statuscode = 17 where name = 'b';
    UPDATE 1

- Make sure only a single package was changed.::

    pkgdb=# COMMIT;

    pkgdb=# select id, packageid, collectionid, owner, statuscode from packagelisting where packageid = 6618;
      id   | packageid | collectionid | owner  | statuscode
    -------+-----------+--------------+--------+-----------
     42552 |      6618 |           19 | 101437 |          3
     38845 |      6618 |           15 | 101437 |          3
     38846 |      6618 |           14 | 101437 |          3
     38844 |      6618 |            8 | 101437 |          3
    (rows 4)

- Make sure the output here looks correct (packageid is all the same, etc).
- You'll also need the ids for later::

     pkgdb=# BEGIN;
     pkgdb=# update packagelisting set statuscode = 17  where packageid = 6618;
     UPDATE 4
     -- Make sure the same number of rows were committed as you saw before.
     pkgdb=# COMMIT;

     pkgdb=# select * from personpackagelisting where packagelistingid in (38844, 38846, 38845, 42552);
      id | userid | packagelistingid.
      ----+--------+------------------
      (0 rows)

- In this case there are no comaintainers so we don't have to do anymore.  If
  there were we'd have to treat them like groups handled next::

     pkgdb=# select * from grouppackagelisting where packagelistingid in (38844, 38846, 38845, 42552);
       id   | groupid | packagelistingid.
     -------+---------+------------------
      39229 |  100300 |            38844
      39230 |  107427 |            38844
      39231 |  100300 |            38845
      39232 |  107427 |            38845
      39233 |  100300 |            38846
      39234 |  107427 |            38846
      84481 |  107427 |            42552
      84482 |  100300 |            42552
     (8 rows)

      pkgdb=# select * from grouppackagelistingacl where grouppackagelistingid in (39229, 39230, 39231, 39232, 39233, 39234, 84481, 84482);

- The results of this are usually pretty long. so I've omitted everything but the rows
  (24 rows)
- For groups it's typically 3 (one for each of commit, build, and checkout) *
- number of grouppackagelistings.  In this case, that's 24 so this matches our expectations.::

    pkgdb=# BEGIN;
    pkgdb=# update grouppackagelistingacl set statuscode = 13 where grouppackagelistingid in (39229, 39230, 39231, 39232, 39233, 39234, 84481, 84482);

- Make sure only the number of rows you saw before were updated::

    pkgdb=# COMMIT;

  If the package has existed long enough to have been added to koji, run
  something like the following to "retire" it in koji.::

    koji block-pkg dist-f12 PKGNAME

Add a new release
=================

To add a new Fedora Release, ssh to db02 and do this::

  sudo -u postgres psql pkgdb

- This adds the release for Package ACLs::

    insert into collection (name, version, statuscode, owner, koji_name) values('Fedora', '13', 1, 'jkeating', 'dist-f13');
    insert into branch select id, 'f13', '.fc13', Null, 'f13' from collection where name = 'Fedora' and version = '13';

- If this is for mass branching we probably need to advance the branch information for devel as well.::

    update branch set disttag = '.fc14' where collectionid = 8;

- This adds the new release's repos for the App DB::

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-i386', 'Fedora 13 - i386', 'development/13/i386/os', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-i386-d', 'Fedora 13 - i386 - Debug', 'development/13/i386/debug', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-i386-tu', 'Fedora 13 - i386 - Test Updates', 'updates/testing/13/i386/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-i386-tud', 'Fedora 13 - i386 - Test Updates Debug', 'updates/testing/13/i386/debug/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-x86_64', 'Fedora 13 - x86_64', 'development/13/x86_64/os', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-x86_64-d', 'Fedora 13 - x86_64 - Debug', 'development/13/x86_64/debug', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-x86_64-tu', 'Fedora 13 - x86_64 - Test Updates', 'updates/testing/13/x86_64/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

    insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-13-x86_64-tud', 'Fedora 13 - x86_64 - Test Updates Debug', 'updates/testing/13/x86_64/debug/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '13';

Update App DB for a release going final
=======================================

When a Fedora release goes final, the repositories for it change where
they live. The repo definitions allow the App browser to sync information
from the yum repositories. The PackageDB needs to be updated for the new
areas::

   BEGIN;
   insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-14-i386-u', 'Fedora 14 - i386 - Updates', 'updates/14/i386/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '14';
   insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-14-i386-ud', 'Fedora 14 - i386 - Updates Debug', 'updates/14/i386/debug/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '14';
   update repos set url='releases/14/Everything/i386/os/' where shortname = 'F-14-i386';
   update repos set url='releases/14/Everything/i386/debug/' where shortname = 'F-14-i386-d';

   insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-14-x86_64-u', 'Fedora 14 - x86_64 - Updates', 'updates/14/x86_64/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '14';
   insert into repos (shortname, name, url, mirror, active, collectionid) select  'F-14-x86_64-ud', 'Fedora 14 - x86_64 - Updates Debug', 'updates/14/x86_64/debug/', 'http://download.fedoraproject.org/pub/fedora/linux/', true, c.id  from collection as c where c.name = 'Fedora' and c.version = '14';
   update repos set url='releases/14/Everything/x86_64/os/' where shortname = 'F-14-x86_64';
   update repos set url='releases/14/Everything/x86_64/debug/' where shortname = 'F-14-x86_64-d';
   COMMIT;

Orphaning all the packages for a user
=====================================

This can be done in the database if you don't want to send email::

   $ ssh db02
   $ sudo -u postgres psql pkgdb
   pkgdb> select * from packagelisting where owner = 'xulchris';
   pkgdb> -- Check that the list doesn't look suspicious.... There should be a  record for every fedora release * package
   pkgdb> BEGIN;
   pkgdb> update packagelisting set owner = 'orphan', statuscode = 14 where owner = 'xulchris';
   pkgdb> -- If the right number of rows were changed
   pkgdb> COMMIT;

.. note::
   Note that if you do it via pkgdb-client or the python-fedora API instead,
   you'll want to only orphan the packages on non-EOL branches that exist to
   cut down on the amount of email that's sent. That entails figuring out
   what branches you need to do this on.
