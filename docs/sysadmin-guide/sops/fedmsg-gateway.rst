.. title: fedmsg-gateway SOP
.. slug: infra-fedmsg-gateway
.. date: 2012-10-31
.. taxonomy: Contributors/Infrastructure

==================
fedmsg-gateway SOP
==================

Outgoing raw ZeroMQ message stream.

.. note::  see also: fedmsg-websocket

Contact Information
===================

Owner:
  Messaging SIG, Fedora Infrastructure Team
Contact:
  #fedora-apps, #fedora-admin, #fedora-noc
Servers:
  busgateway01, proxy0*
Purpose:
  Expose raw ZeroMQ messages outside the FI environment.

Description
===========

Users outside of Fedora Infrastructure can listen to the production message
bus by connecting to specific addresses.  This is required for local users to
run their own hubs and message processors ("Consumers").  It is also
required for user-facing tools like fedmsg-notify to work.

The specific public endpoints are:

production
  tcp://hub.fedoraproject.org:9940
staging
  tcp://stg.fedoraproject.org:9940

fedmsg-gateway, the daemon running on busgateway01, is listening to the FI
production fedmsg bus and will relay every message that it receives out to a
special ZMQ pub endpoint bound to port 9940.  haproxy mediates connections
to the fedmsg-gateway daemon.

Connection Flow
===============

Clients connect through haproxy on proxy0*:9940 are redirected to
busgateway0*:9940.  This can be found in the haproxy.cfg entry for
``listen fedmsg-raw-zmq 0.0.0.0:9940``.

This is different than the apache reverse proxy pass setup we have for the
app0* and packages0* machines.  *That* flow looks something like this::

    Client -> apache(proxy01) -> haproxy(proxy01) -> apache(app01)

The flow for the raw zmq stream provided by fedmsg-gateway looks something
like this::

    Client -> haproxy(proxy01) -> fedmsg-gateway(busgateway01)

haproxy is listening on a public port.

At the time of this writing, haproxy does not actually load balance zeromq
session requests across multiple busgateway0* machines, but there is nothing
stopping us from adding them.  New hosts can be added in ansible and pressed
from busgateway01's template.  Add them to the fedmsg-raw-zmq listen in
haproxy's config and it should Just Work.

Increasing the Maximum Number of Concurrent Connections
=======================================================

HTTP requests are typically very short (a few seconds at most).  This
means that the number of concurrent tcp connections we require for most
of our services is quite low (1024 is overkill).  ZeroMQ tcp connections,
on the other hand, are expected to live for quite a long time.
Consequently we needed to scale up the number of possible concurrent tcp
connections.

All of this is in ansible and should be handled for us automatically if we
bring up new nodes.

- The pam_limits user limit for the fedmsg user was increased from
  1024 to 160000 on busgateway01.
- The pam_limits user limit for the haproxy user was increased from
  1024 to 160000 on the proxy0* machines.
- The zeromq High Water Mark (HWM) was increased to 160000 on
  busgateway01.
- The maximum number of connections allowed was increased in haproxy.cfg.

Nagios
======

New nagios checks were added for this that check to see if the number of
concurrent connections through haproxy is approaching the maximum number
allowed.

You can check these numbers by hand by inspecting the haproxy web interface:
https://admin.fedoraproject.org/haproxy/proxy1#fedmsg-raw-zmq

Look at the "Sessions" section.  "Cur" is the current number of sessions
versus "Max", the maximum number seen at the same time and "Limit", the
maximum number of concurrent connections allowed.

RHIT
====

We had RHIT open up port 9940 special to proxy01.phx2 for this.
