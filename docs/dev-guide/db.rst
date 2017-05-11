=========
Databases
=========

We use PostgreSQL throughout Fedora Infrastructure.


Bi-directional Replication
==========================

`Bi-directional replication`_ (BDR) is a project that adds asynchronous multi-
master logical replication to PostgreSQL. Fedora has a PostgreSQL deployment with
BDR enabled. In Fedora, only one master is written to at any time.

Applications are not required to use the BDR-enabled database, but it is
encouraged since it provides redundancy and more flexibility for the system
administrators.

Applications need to take several things into account when considering whether
or not to use BDR.


Primary Keys
------------

All tables need to have primary keys.


Conflicts
---------

BDR does not use any consensus algorithm or locking between nodes so writing to
multiple masters can result in `conflicts`_. There are several types of conflicts
that can occur, and applications should carefully consider each one and be
prepared to handle them. Some conflicts are handled automatically, while others can
result in a deadlock that requires manual intervention.


Global DDL Lock
---------------

BDR uses a `global DDL lock`_ (across all PostgreSQL nodes)  for DDL changes,
which applications must explicitly acquire prior to emitting DDL statements.

This can be done in Alembic by modifying the ``run_migrations_offline`` and
``run_migrations_online`` functions in ``env.py`` to emit the SQL when
connecting to the database. An example of the ``run_migrations_offline``::

    def run_migrations_offline():
        """Run migrations in 'offline' mode.

        This requires a configuration options since it's not known whether the
        target database is a BDR cluster or not. Alternatively, you can simply
        add the SQL to the script manually and not bother with a setting.
        """
        url = config.get_main_option("sqlalchemy.url")
        context.configure(url=url)

        with context.begin_transaction():
            # If the configuration indicates this script is for a Postgres-BDR database,
            # then we need to acquire the global DDL lock before migrating.
            postgres_bdr = config.get_main_option('offline_postgres_bdr')
            if postgres_bdr is not None and postgres_bdr.strip().lower() == 'true':
                _log.info('Emitting SQL to allow for global DDL locking with BDR')
                context.execute('SET LOCAL bdr.permit_ddl_locking = true')
            context.run_migrations()

An example of the ``run_migrations_online`` function::

    def run_migrations_online():
        """Run migrations in 'online' mode.

        This auto-detects when it's run against a Postgres-BDR system.
        """
        engine = engine_from_config(
            config.get_section(config.config_ini_section),
            prefix='sqlalchemy.',
            poolclass=pool.NullPool)

        connection = engine.connect()
        context.configure(
            connection=connection,
            target_metadata=target_metadata)

        try:
            try:
                connection.execute('SHOW bdr.permit_ddl_locking')
                postgres_bdr = True
            except exc.ProgrammingError:
                # bdr.permit_ddl_locking is an unknown option, so this isn't a BDR database
                postgres_bdr = False
            with context.begin_transaction():
                if postgres_bdr:
                    _log.info('Emitting SQL to allow for global DDL locking with BDR')
                    connection.execute('SET LOCAL bdr.permit_ddl_locking = true')
                context.run_migrations()
        finally:
            connection.close()

Be aware that long-running migrations will hold the global lock for the entire
migration and while the global lock is held by a node, no other nodes may perform
any DDL or make any changes to rows.


DDL Restrictions
----------------

BDR has a set of `DDL Restrictions`_. Some of the restrictions are easily worked
around by performing the task in several steps, while others are simply not
available.

.. _Bi-directional replication: http://bdr-project.org/docs/stable/index.html
.. _Conflicts: http://bdr-project.org/docs/stable/conflicts.html
.. _Global DDL Lock: bdr-project.org/docs/stable/ddl-replication-advice.html
.. _DDL Restrictions:
    http://bdr-project.org/docs/stable/ddl-replication-statements.html
    #DDL-REPLICATION-PROHIBITED-COMMANDS
