.. title: Koji Infrastructure SOP
.. slug: infra-koji
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

=======================
Koji Infrastructure SOP
=======================

.. note::
   We are transitioning from two buildsystems, koji for Fedora and plague for
   EPEL, to just using koji. This page documents both.

Koji and plague are our buildsystems. They share some of the same machines
to do their work.

Contents
========

1. Contact Information
2. Description
3. Add packages into Buildroot
4. Troubleshooting and Resolution

  1. Restarting Koji
  2. kojid won't start or some builders won't connect
  3. OOM (Out of Memory) Issues

    1. Increase Memory
    2. Decrease weight

  4. Disk Space Issues

5. Should there be mention of being sure filesystems in chroots are
unmounted before you delete the chroots?

Contact Information
===================

Owner
	 Fedora Infrastructure Team

Contact
	 #fedora-admin, sysadmin-build group

Persons
	 mbonnet, dgilmore, f13, notting, mmcgrath, SmootherFrOgZ

Location
	 Phoenix

Servers
  - koji.fedoraproject.org
  - buildsys.fedoraproject.org
  - xenbuilder[1-4] 
  - hammer1, ppc[1-4]

Purpose
	 Build packages for Fedora.

Description
===========

Users submit builds to koji.fedoraproject.org or
buildsys.fedoraproject.org. From there it gets passed on to the builders.

.. important::
   At present plague and koji are unaware of each other. A result of this may
   be an overloaded builder. A easy fix for this is not clear at this time

Add packages into Buildroot
===========================

Some contributors may have the need to build packages against fresh built
packages which are not into buildroot yet. Koji has override tags as a
Inheritance to the build tag in order to include them into buildroot which
can be set by::

  koji tag-pkg dist-$release-override <package_nvr>

Troubleshooting and Resolution
==============================

Restarting Koji
---------------

If for some reason koji needs to be restarted, make sure to restart the
koji master first, then the builders. If the koji master has been down for
a short enough time the builders do not need to be restarted.::

  service httpd restart
  service kojira restart
  service kojid restart

.. important::   
  If postgres becomes interrupted in some way, koji will need to be
  restarted. As long as the koji master daemon gets restarted the builders
  should reconnect automatically. If the db server has been restarted and
  the builders don't seem to be building, restart their daemons as well.

kojid won't start or some builders won't connect
------------------------------------------------

In the event that some items are able to connect to koji while some are
not, please make sure that the database is not filled up on connections.
This is common if koji crashes and the db connections aren't properly
cleared. Upon restart many of the connections are full so koji cannot
reconnect. Clearing old connections is easy, guess about how long it the
new koji has been up and pick a number of minutes larger then that and
kill those queries. From db3 as postgres run::

  echo "select procpid from pg_stat_activity where usename='koji' and now() - query_start \
  >= '00:40:00' order by query_start;" | psql koji | grep "^  " | xargs kill

OOM (Out of Memory) Issues
--------------------------

Out of memory issues occur from time to time on the build machines. There
are a couple of options for correction. The first fix is to just restart
the machine and hope it was a one time thing. If the problem continues
please choose from one of the following options.

Increase Memory
```````````````

The xen machines can have memory increased on their corresponding xen
hosts. At present this is the table:

+----------+-------------+
| xen3     | xenbuilder1 |
+----------+-------------+
| xen4     | xenbuilder2 |
+----------+-------------+
| disabled | xenbuilder3 |
+----------+-------------+
| xen8     | xenbuilder4 |
+----------+-------------+

Edit ``/etc/xen/xenbuilder[1-4]`` and add more memory.

Decrease weight
```````````````

Each builder has a weight as to how much work can be given to it.
Presently the only way to alter weight is actually changing the database
on db3::

  $ sudo su - postgres
  -bash-2.05b$ psql koji
  koji=# select * from host limit 1;
  id | user_id |          name          |  arches   | task_load | capacity | ready | enabled
  ---+---------+------------------------+-----------+-----------+----------+-------+---------
  6  |     130 | ppc3.fedora.redhat.com | ppc ppc64 |       1.5 |        4 | t     | t
  (1 row)
  koji=# update host set capacity=2 where name='ppc3.fedora.redhat.com';

Simply update capacity to a lower number.

Disk Space Issues
------------------

The builders use a lot of temporary storage. Failed builds also get left
on the builders, most should get cleaned but plague does not. The easiest
thing to do is remove some older cache dirs.

Step one is to turn off both koji and plague::

  /etc/init.d/plague-builder stop
  /etc/init.d/kojid stop

Next check to see what file system is full::

  df -h

.. important::  
   If any one of the following directories is full, send an outage
   notification as outlined in: [62]Infrastructure/OutageTemplate to the
   fedora-infrastructure-list and fedora-devel-list, then contact Mike
   McGrath

    - /mnt/koji
    - /mnt/ntap-fedora1/scratch
    - /pub/epel
    - /pub/fedora

Typically just / will be full. The next thing to do is determine if we
have any extremely large builds left on the builder. Typical locations
include /var/lib/mock and /mnt/build (/mnt/build actually is on the local
filesystem)::

  du -sh /var/lib/mock/* /mnt/build/*

``/var/lib/mock/dist-f8-build-10443-1503``
  classic koji build
``/var/lib/mock/fedora-6-ppc-core-57cd31505683ef1afa533197e91608c5a2c52864``
  classic plague build

If nothing jumps out immediately, just start deleting files older than one
week. Once enough space has been freed start koji and plague back up::

  /etc/init.d/plague-builder start
  /etc/init.d/kojid start

Unmounting
----------

.. warning:: 
  Should there be mention of being sure filesystems in chroots 
  are unmounted before you delete the chroots?

  Res ipsa loquitur.

