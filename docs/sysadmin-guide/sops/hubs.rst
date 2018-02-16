.. title: Fedora Hubs SOP
.. slug: infra-hubs
.. date: 2018-02-16
.. taxonomy: Contributors/Infrastructure

===============
Fedora Hubs SOP
===============


Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main, sysadmin-tools, sysadmin-hosted

Location
	?

Servers
	<prod-srv-hostname>, <stg-srv-hostname>, hubs-dev.fedorainfracloud.org

Purpose
	Contributor and team portal.


Description
===========

Fedora Hubs aggregates user and team activity throughout the Fedora
infrastructure (and elsewhere) to show what a user or a team is doing. It helps
new people find a place to contribute.


Components
----------

Fedora Hubs has the following components:

- a SQL database like PostgreSQL (in the Fedora infra we're using the shared
  database).
- a Redis server that is used as a message bus (it is not critical if the
  content is lost). System service: ``redis``.
- a MongoDB server used to store the contents of the activity feeds. It's JSON
  data, limited to 100 entries per user or group. Service: ``mongod``.
- a Flask-based WSGI server that will also serve the JS front end as static
  files. System service: ``fedora-hubs-webapp``.
- a Fedmsg listener that receives messages from the fedmsg bus and puts them in
  Redis. System service: ``fedmsg-hub``.
- a set of "triage" workers that pull the raw messages from Redis, process them
  using SQL queries and puts work items in another Redis queue. System service:
  ``fedora-hubs-triage@``.
- a set of "worker" daemons that pull from this other Redis queue, work on the
  items by making SQL queries and external HTTP requests (to Github for
  example), and put reload notifications in the SSE Redis queue. They also
  access the caching system, which can be local files or memcached. System
  service: ``fedora-hubs-worker@``.
- The SSE server (Twisted-based) that pulls from that Redis queue and sends
  reload notifications to the connected browsers. It handles long-lived HTTP
  connection but there is little activity: only the notifications and a
  "keepalive ping" message every 30 seconds to every connected browser.
  System service: ``fedora-hubs-sse``.


Managing the services
=====================

Restarting all the services::

    systemctl restart fedmsg-hub fedora-hubs-\*

By default, 4 ``triage`` daemons and 4 ``worker`` daemons are enabled. To add
another ``triage`` daemon and another ``worker`` daemon, you can run::

    systemctl enable --now fedora-hubs-triage@5.service
    systemctl enable --now fedora-hubs-worker@5.service

It is not necessary to have the same number of ``triage`` and ``worker``
daemons, in fact it is expected that more ``worker`` than ``triage`` daemons
will be necessary, as they do more time-consuming work.


Hubs-specific operations
========================

Other Hubs-specific operations are done using the `fedora-hubs` command::

    $ fedora-hubs
    Usage: fedora-hubs [OPTIONS] COMMAND [ARGS]...

    Options:
      --help  Show this message and exit.

    Commands:
      cache  Cache-related operations.
      fas    FAS-related operations.
      run    Run daemon processes.

Manipulating the cache
----------------------
The ``cache`` subcommand is used to do cache-related operations::

    $ fedora-hubs cache
    Usage: fedora-hubs cache [OPTIONS] COMMAND [ARGS]...

      Cache-related operations.

    Options:
      --help  Show this message and exit.

    Commands:
      clean     Clean the specified WIDGETs (id or name).
      coverage  Check the cache coverage.
      list      List widgets for which there is cached data.

For example, to check the cache coverage::

    $ fedora-hubs cache coverage
    107 cached values found, 95 are missing.
    52.97 percent cache coverage.

The cache coverage value is an interesting metric that could be used in a
Nagios check. A value below 50% could be considered as significant of
application slowdowns and could thus generate a warning.

Interacting with FAS
--------------------
The ``fas`` subcommand is used to get information from FAS::

    $ fedora-hubs fas
    Usage: fedora-hubs fas [OPTIONS] COMMAND [ARGS]...

      FAS-related operations.

    Options:
      --help  Show this message and exit.

    Commands:
      create-team  Create the team hub NAME from FAS.
      sync-teams   Sync all the team hubs NAMEs from FAS.

To add a new team hub for a FAS group, run::

    $ fedora-hubs fas create-team <fas-group-name>

