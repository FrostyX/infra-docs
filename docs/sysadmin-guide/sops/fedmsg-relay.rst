.. title: fedmsg-relay SOP
.. slug: infra-fedmsg-relay
.. date: 2012-10-31
.. taxonomy: Contributors/Infrastructure

================
fedmsg-relay SOP
================

Bridge ephemeral scripts into the fedmsg bus.

Contact Information
===================

Owner
	Messaging SIG, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-admin, #fedora-noc
Servers
	app01
Purpose
	Bridge ephemeral bash and python scripts into the fedmsg bus.

Description
===========

fedmsg-relay is running on app01, which is a bad choice.  We should look to
move it to a more isolated place in the future.  busgateway01 would be a
better choice.

"Ephemeral" scripts like ``pkgdb2branch.py``, the post-receive git hook on
pkgs01, and anywhere fedmsg-logger is used all depend on fedmsg-relay.
Instead of emitting messages "directly" to the rest of the bus, they use
fedmsg-relay as an intermediary.

Check that fedmsg-relay is running by looking for it in the process list.
You can restart it in the standard way with ``sudo service fedmsg-relay
restart``.  Check for its logs in ``/var/log/fedmsg/fedmsg-relay.log``

Ephemeral scripts know where the fedmsg-relay is by looking for the
relay_inbound and relay_outbound values in the global fedmsg config.

But What is it Doing?  And Why?
===============================

The fedmsg bus is designed to be "passive" in its normal operation.  A
mod_wsgi process under httpd sets up its fedmsg publisher socket to
passively emit messages on a certain port.  When some other service wants
to receive these messages, it is up to that service to know where mod_wsgi
is emitting and to actively connect there.  In this way, emitting is passive
and listening is active.

We get a problem when we have a one-off or "ephemeral" script that is not a
long-running process -- a script like pkgdb2branch which is run when a user
runs it and which ends shortly after.  Listeners who want these scripts
messages will find that they are usually not available when they try to
connect.

To solve this problem, we introduced the "fedmsg-relay" daemon which is a
kind of "passive"-to-"passive" adaptor.  It binds to an outbound port on one
end where it will publish messages (like normal) but it also binds to an
another port where it listens passively for inbound messages.  Ephemeral
scripts then actively connect to the passive inbound port of the
fedmsg-relay to have their payloads echoed on the bus-proper.

See http://fedmsg.readthedocs.org/en/latest/topology/ for a diagram.
