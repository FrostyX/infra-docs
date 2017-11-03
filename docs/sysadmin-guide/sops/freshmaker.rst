.. title: Freshmaker SOP
.. slug: infra-freshmaker
.. date: 2017-10-07
.. taxonomy: Contributors/Infrastructure

==============
Freshmaker SOP
==============

.. note::
   Freshmaker is very new and changing rapidly.  We'll try to keep this up to
   date as best we can.

Freshmaker is a service that watches message bus activity and tries to rebuild
*compound* artifacts when their constituent pieces change.

Contact Information
===================

Owner
	 Factory2 Team, Release Engineering Team, Infrastructure Team

Contact
	 #fedora-modularity, #fedora-admin, #fedora-releng

Persons
	 jkaluza, cqi, qwan, sochotni, threebean

Location
	 Phoenix

Public addresses
  - freshmaker.fedoraproject.org

Servers
  - freshmaker-frontend0[1-2].phx2.fedoraproject.org
  - freshmaker-backend01.phx2.fedoraproject.org

Purpose
	 Rebuild compound artifacts.  See description for more detail.

Description
===========

See also http://fedoraproject.org/wiki/Infrastructure/Factory2/Focus/Freshmaker
for some of the original (old) thinking on Freshmaker.

As per the summary above, Freshmaker is a bus-oriented system that watches for
changes to smaller pieces of content, and triggers rebuilds of larger pieces of
content.

It doesn't do the actual *builds* itself, but instead requests rebuilds in our
existing build systems.

It handles a number of different content types.  In Fedora, we would
like to roll out rebuilds in the following order:

Module Builds
-------------

When a spec file changes on a particular dist-git branch, trigger rebuilds of
all modules that declare dependencies on that rpm branch.

Consider the *traditional workflow* today.  You make a patch to the `f27` of your
package, and you know you need to build that patch for f27, and then later
submit an update for this single build.  Packagers know what to do.

Consider the *modular workflow*.  You make a patch to the `2.2` branch of your
package, but now, which modules do you rebuild?  Maybe you had one in mind that
you wanted to fix, but are there others that you forgot about -- that you don't
even know about?  Kevin could maintain a module that pulls in my rpm branch and
he never told me.  Even if he did, I have to now maintain a list of modules
that depend on my rpm, and request rebuilds of them everytime I patch my .spec
file.  This is unmanageable.

Freshmaker deals with this by watching the bus for dist-git fedmsg messages.
When it sees a change on a branch, it looks up the list of modules that depend
on that branch, and requests rebuilds of them in the MBS.

Container Slow Flow
-------------------
When a traditional rpm or modular rpm is *shipped stable*, this trigger rebuilds
of all containers that ever included previous versions of this rpm.

This applies to both modular and non-modular contexts.  Today, you build an rpm
that fixes a CVE, but *some other person* maintains a container that includes
your RPM.  Maybe they never told you about this.  Maybe they didn't notice your
CVE fix.  Their container will remain outdated and vulnerable.. forever?

Freshmaker deals with this by watching the bus for dist-git messages about rpms
being shipped to the stable updates repo.  When they're shipped, it looks up
all containers that ever included pervious versions of the rpm in question, and
it triggers rebuilds of them.

*Waiting* until the rpm ships to stable is *necessary* because the container
build process doesn't know about unshipped content.  This is how containers are
built manually today, and it is annoying.  Which brings us to the more
complicated...

Container Fast Flow
-------------------
When a traditional rpm or modular rpm is *signed*, generate a repo containing
it and rebuild all containers that ever included that rpm before. This is the
better version of the slow flow, but is more complicated so we're deferring it
until after we've proved the first two cases out.

Freshmaker will do this by requesting an interim build repo from ODCS (the On
Demand Compose Service).  ODCS can be given the appropriate koji tag and will
produce a repo of (pre-signed) rpms.  Freshmaker will request a rebuild of the
container and will pass the ODCS repo url in.  This gives us an auditable trail
of disposable repos.

Systems
=======

There is a frontend and a backend.

Everything in the previous section describes the backend behavior.

The frontend exists to provide an HTTP API that can be queried to find out the
status of the backend:  What is it doing?  What is it planning to do?  What has
it done already?

Observing Freshmaker Behavior
=============================

There is currently no command line tool to query Freshmaker, but Freshmaker
provides REST API which can be used to observe Freshmaker behavior. This is
available at the following URLs:

- https://freshmaker.fedoraproject.org/api/1/events
- https://freshmaker.fedoraproject.org/api/1/builds

The first `/events` URL should return a list of events that Freshmaker has
noticed, recorded, and is handling.  Handled events should produce associated
builds.

The second `/builds` URL should return a list of builds that Freshmaker has
submitted and is monitoring.  Each build should be traceable back to the event
that triggered it.

Logs
====

The frontend logs are on freshmaker-frontend0[1-2] in ``/var/log/httpd/error_log``.

The backend logs are on freshmaker-backend01.  Look in the journal for the
`fedmsg-hub` service.

Upgrading
=========

The package in question is `freshmaker`.  Please use the
`playbooks/manual/upgrade/freshmaker.yml` playbook.

Things that could go wrong
==========================

TODO.  We don't know yet.  Probably lots of things.
