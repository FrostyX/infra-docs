.. title: On Demand Compose Service SOP
.. slug: infra-odcs
.. date: 2017-09-19
.. taxonomy: Contributors/Infrastructure

==============================
On Demand Compose Service SOP
==============================

.. note::
   The ODCS is very new and changing rapidly.  We'll try to keep this up to date
   as best we can.

The ODCS is a service generating temporary compose from Koji tag(s) using
Pungi.

Contact Information
===================

Owner
	 Factory2 Team, Release Engineering Team, Infrastructure Team

Contact
	 #fedora-modularity, #fedora-admin, #fedora-releng

Persons
	 jkaluza, cqi, qwan, threebean

Location
	 Phoenix

Public addresses
  - odcs.fedoraproject.org

Servers
  - odcs-frontend0[1-2].phx2.fedoraproject.org
  - odcs-backend01.phx2.fedoraproject.org

Purpose
	 Generate temporary compose from Koji tag(s) using Pungi.

Description
===========

ODCS clients submit request for a compose to odcs.fedoraproject.org. The
requests are submitted using python2-odcs-client Python module or just using
plain JSON.

The request contains all the information needed to build a compose:

- source type: Type of compose source, for example "tag" or "module"
- source: Name of Koji tag or list of modules defined by name-stream-version.
- packages: List of packages to include in a compose.
- seconds to live: Number of seconds after which the compose is removed from
  the filesystem and is marked as "removed".
- flags: Various flags further defining the compose - for example the "no_deps"
  flag saying that the `packages` dependencies should not be included in a
  compose.

The request is received by the ODCS flask app running on odcs-frontend nodes.
The frontend does input validation of the request and then adds the compose
request to database with "wait" state and sends fedmsg message about this
event. The compose request gets its unique id which can be used by a client
to query its status using frontend REST API.

The odcs-backend node then handles the compose requests in "wait" state and
starts generating the compose using the Pungi tool. It does so by generating
all the configuration files for Pungi and executing "pungi" executable.
Backend also changes the compose request status to "generating" and sends
fedmsg message about this event.

The number of concurrent pungi processes can be set using the
`num_concurrent_pungi` variable in ODCS configuration file.

The output directory for a compose is shared between frontend and backend node.
Once the compose is generated, the backend changes the status of compose
request to "done" and again sends fedmsg message about this event.

The shared directory with a compose is available using httpd on the frontend
node and ODCS client can access the generated compose. By default this is on
https://odcs.fedoraproject.org/composes/ URL.

If the compose generation goes wrong, the backend changes the state of the
compose request to "failed" and again sends fedmsg message about this event.
The "failed" compose is still available for `seconds to live` time in the
shared directory for further examination of pungi logs if needed.

After the `seconds to live` time, the backend node removes the compose from
filesystem and changes the state of compose request to "removed".

If there are compose requests for the very same composes, the ODCS will reuse
older compose instead of generating new one and points the new compose to
older one.

The "removed" compose can be renewed by a client to generate the same compose
as in the past. The `seconds to live` attribute of a compose can be extended
by a client when needed.

Observing ODCS Behavior
======================

There is currently no command line tool to query ODCS, but ODCS provides REST
API which can be used to observe the ODCS behavior. This is available on
https://odcs.fedoraproject.org/api/1/composes.

The API can be filtered by following keys entered as HTTP GET variables:

- owner
- source_type
- source
- state

It is also possible to see all the current composes in the compose output
directory, which is available on the frontend on
https://odcs.fedoraproject.org/composes.


Removing compose before its expiration time
===========================================

Members of FAS group defined in the `admins` section of ODCS configuration
can remove any compose by sending DELETE request to following URL:

https://odcs.fedoraproject.org/api/1/composes/$compose_id

Logs
====

The frontend logs are on odcs-frontend0[1-2] in ``/var/log/httpd/error_log``
or ``/var/log/httpd/ssl_error_log``.

The backend logs are on odcs-backend01.  Look in the journal for the
`odcs-backend` service.

Upgrading
=========

The package in question is `odcs-server`.  Please use the
`playbooks/manual/upgrade/odcs.yml` playbook.

Things that could go wrong
==========================

Not enough space on shared volume
---------------------------------

In case there are too many composes, member of FAS group defined in the
ODCS configuration file `admins` section should:

- Remove the oldest composes to get some free space immediatelly. List of such
  composes can be found on https://odcs.fedoraproject.org/composes/ by sorting
  by Last modified fields.
- Decrease the `max_seconds_to_live` in ODCS configuration file.
