.. title: websocket SOP
.. slug: infra-websocket
.. date: 2012-10-31
.. taxonomy: Contributors/Infrastructure

=============
websocket SOP
=============

websocket communication with Fedora apps.

see-also:  ``fedmsg-gateway.txt``

Contact Information
===================

Owner
  Messaging SIG, Fedora Infrastructure Team
Contact
  #fedora-apps, #fedora-admin, #fedora-noc
Servers
  busgateway01, proxy0*, app0*
Purpose
  Expose a websocket server for FI apps to use

Description
===========

WebSocket is a protocol (an extension of HTTP/1.1) by which client web
browsers can establish full-duplex socket communications with a server --
the "real-time web".

In our case, webapps served from app0* and packages0* will include
javascript code instructing client browsers to establish a second connection
to our WebSocket server.  They point browsers to the following addresses:

production
  wss://hub.fedoraproject.org:9939
staging
  wss://stg.fedoraproject.org:9939

The websocket server itself is a fedmsg-hub daemon running on busgateway01.
It is configured to enable its websocket server component in the presence of
certain configuration values.

haproxy mediates connections to the fedmsg-hub websocket server daemon.
An stunnel daemon provides SSL support.

Connection Flow
===============

The connection flow is much the same as in the fedmsg-gateway.txt SOP, but
is somewhat more complicated.

"Normal" HTTP requests to our app servers traverse the following chain::

  Client -> apache(proxy01) -> haproxy(proxy01) -> apache(app01)

The flow for a websocket requests looks something like this::

  Client -> stunnel(proxy01) -> haproxy(proxy01) -> fedmsg-hub(busgateway01)

stunnel is listening on a public port, negotiates the SSL connection, and
redirects the connection to haproxy who in turn hands it off to
the fedmsg-hub websocket server listening on busgateway01.

At the time of this writing, haproxy does not actually load balance zeromq
session requests across multiple busgateway0* machines, but there is nothing
stopping us from adding them.  New hosts can be added in ansible and pressed
from busgateway01's template.  Add them to the fedmsg-websockets listen in
haproxy's config and it should Just Work.

RHIT
====

We had RHIT open up port 9939 special to proxy01.phx2 for this.
