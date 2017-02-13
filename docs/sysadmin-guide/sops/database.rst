.. title: Database Infrastructure SOP
.. slug: infra-database
.. date: 2016-09-24
.. taxonomy: Contributors/Infrastructure

===========================
Database Infrastructure SOP
===========================

Our database servers provide database storage for many of our apps.

Contents

1. Contact Information
2. Description
3. Creating a New Postgresql Database
4. Troubleshooting and Resolution

	1. Connection issues
	2. Some useful queries

		1. What queries are running
		2. Seeing how "dirty" a table is
		3. XID Wraparound

	3. Restart Procedure

		1. Koji

		2. Bodhi

5. Note about TurboGears and MySQL
6. Restoring from backups or specific dbs


Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main, sysadmin-dba group

Location
	Phoenix

Servers
	sb01, db03, db-fas01, db-datanommer02, db-koji01, db-s390-koji01, db-arm-koji01, db-ppc-koji01, db-qa01, dbqastg01

Purpose
	Provides database connection to many of our apps.

Description
===========

db01, db03 and db-fas01 are our primmary servers.
db01 and db-fas01 run PostgreSQL.
db03 contain mariadb.
db-koji01, db-s390-koji01, db-arm-koji01, db-ppc-koji01 contain secondary kojis.
db-qa01 and db-qastg01 contain taskotron and resultsdb.
db-datanommer02 contains all storage messages from postgresql database.


Creating a New Postgresql Database
==================================

Creating a new database on our postgresql server isn't hard but there's
several steps that should be taken to make the database server as secure
as possible.

We want to separate the database permissions so that we don't have the
user/password combination that can do anything it likes to the database on
every host (the webapp user can usually do a lot of things even without those
extra permissions but every little bit helps).

Say we have an app called "raffle".  We'd have three users:

* raffleadmin: able to make any changes they want to this particular
  database.  It should not be used in day to day but only for things
  like updating the database schema when an update occurs.
  We could very likely disable this account in the db whenever we are not
  using it.
* raffleapp: the database user that the web application uses.  This will
  likely need to be able to insert and select from all tables.  It will
  probably need to update most tables as well.  There may be some tables
  that it does *not* need delete on.  It should almost certainly not
  need schema modifying permissions.  (With postgres, it likely also
  needs permission to insert/select on sequences as well).
* rafflereadonly: Only able to read data from tables, not able to modify
  anything.  Sadly, we aren't using this often but it can be useful for
  scripts that need to talk directly to the database without modifying it.

::

  db2 $ sudo -u postgres createuser -P -E NEWDBadmin
  Password: <randomly generated password>
  db2 $ sudo -u postgres createuser -P -E NEWDBapp
  Password: <randomly generated password>
  db2 $ sudo -u postgres createuser -P -E NEWDBreadonly
  Password: <randomly generated password>
  db2 $ sudo -u postgres createdb -E utf8 NEWDB -O NEWDBadmin
  db2 $ sudo -u postgres psql NEWDB
  NEWDB=# revoke all on database NEWDB from public;
  NEWDB=# revoke all on schema public from public;
  NEWDB=# grant all on schema public to NEWDBadmin;
  NEWDB=# [grant permissions to NEWDBapp as appropriate for your app]
  NEWDB=# [grant permissions to NEWDBreadonly as appropriate for a user that
         is only trusted enough to read information]
  NEWDB=# grant connect on database NEWDB to nagiosuser;


If your application needs to have the NEWDBapp and password to connect to
the database, you probably want to add these to ansible as well. Put the
password in the private repo in batcave01. Then use a templatefile to
incorporate it into the config file. See fas.pp for an example.

Troubleshooting and Resolution
==============================

Connection issues
-----------------

There are no known outstanding issues with the database itself. Remember
that every time either database is restarted, services will have to be
restarted (see below).

Some useful queries
-------------------

What queries are running
````````````````````````

This can help you find out what queries are cuurently running on the
server::

  select datname, pid, query_start, backend_start, query from
  pg_stat_activity where state<>'idle' order by query_start;

This can help you find how many connections to the db server are for each
individual database::

  select datname, count(datname) from pg_stat_activity group by datname
  order by count desc;

Seeing how "dirty" a table is
`````````````````````````````

We've added a function from postgres's contrib directory to tell how dirty
a table is. By dirty we mean, how many tuples are active, how many have
been marked as having old data (and therefore "dead") and how much free
space is allocated to the table but not used.::

  \c fas2
  \x
  select * from pgstattuple('visit_identity');
  table_len          | 425984
  tuple_count        | 580
  tuple_len          | 46977
  tuple_percent      | 11.03
  dead_tuple_count   | 68
  dead_tuple_len     | 5508
  dead_tuple_percent | 1.29
  free_space         | 352420
  free_percent       | 82.73
  \x

Vacuum should clear out dead_tuples. Only a vacuum full, which will lock
the table and therefore should be avoided, will clear out free space.

XID Wraparound
``````````````
Find out how close we are to having to perform a vacuum of a database (as
opposed to individual tables of the db). We should schedule a vacuum when
about 50% of the transaction ids have been used (approximately 530,000,000
xids)::

  select datname, age(datfrozenxid), pow(2, 31) - age(datfrozenxid) as xids_remaining
  from pg_database order by xids_remaining;

Information on [61]wraparound

Restart Procedure
=================

If the database server needs to be restarted it should come back on it's
own. Otherwise each service on it can be restarted::

  service mysqld restart
  service postgresql restart

Koji
----

Any time postgreql is restarted, koji needs to be restarted. Please also
see [62]Restarting Koji

Bodhi
-----

Anytime postgresql is restarted Bodhi will need to be restarted no sop
currently exists for this.

TurboGears and MySQL
====================

.. note:: about TurboGears and MySQL

   There's a known bug in TurboGears that causes MySQL clients not to
   automatically reconnect when lost. Typically a restart of the TurboGears
   application will correct this issue.

Restoring from backups or specific dbs.
=======================================

Our backups store the latest copy in /backups/ on each db server.
These backups are created automatically by the db-backup script run fron cron.
Look in /usr/local/bin for the backup script.

To restore partially or completely you need to:

1. setup postgres on a system

2. start postgres/run initdb
    - if this new system running postgres has already run ansible then it will
       have wrong config files in /var/lib/pgsql/data - clear them out before
       you start postgres so initdb can work.
3. grab the backups you need from /backups  - also grab global.sql
    edit up global.sql to only create/alter the dbs you care about

4. as postgres run: ``psql -U postgres -f global.sql``

5. when this completes you can restore each db with (as postgres user)::
      createdb $dbname
      pg_restore -d dbname dbname_backup_file.db

6. restart postgres and check your data.
