.. title: bugzilla2fedmsg SOP
.. slug: infra-bugzilla2fedmsg
.. date: 2016-04-07
.. taxonomy: Contributors/Infrastructure

===================
bugzilla2fedmsg SOP
===================

Receive events from bugzilla over the RH "unified messagebus" and rebroadcast
them over our own fedmsg bus.

Contact Information
-------------------

Owner
	Messaging SIG, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-fedmsg, #fedora-admin, #fedora-noc
Servers
	bugzilla2fedmsg01
Purpose
	Rebroadcast bugzilla events on our bus.

Description
-----------

bugzilla2fedmsg is a small service running as the 'moksha-hub' process which
receives events from bugzilla via the RH "unified messagebus" and rebroadcasts
them to our fedmsg bus.

.. note:: Unlike *all* of our other fedmsg services, this one runs as the
   'moksha-hub' process and not as the 'fedmsg-hub'.

The bugzilla2fedmsg package provides a plugin to the moksha-hub that
connects out over the STOMP protocol to a 'fabric' of JBOSS activemq FUSE
brokers living in the Red Hat DMZ.  We authenticate with a cert/key pair that is
kept in /etc/pki/fedmsg/.  Those brokers should push bugzilla events over
STOMP to our moksha-hub daemon.  When a message arrives, we query bugzilla
about the change to get some 'more interesting' data to stuff in our
payload, then we sign the message using a fedmsg cert and fire it off to the
rest of our bus.

This service has no database, no memcached usage.  It depends on those STOMP
brokers and being able to query bugzilla.rh.com.

Relevant Files
--------------

All managed by ansible, of course:

    STOMP config:   /etc/moksha/production.ini
    fedmsg config:  /etc/fedmsg.d/
    certs:          /etc/pki/fedmsg
    code:           /usr/lib/python2.7/site-packages/bugzilla2fedmsg.py

Useful Commands
---------------

To look at logs, run::

    $ journalctl -u moksha-hub -f

To restart the service, run::

    $ systemctl restart moksha-hub

Internal Contacts
-------------------

If we need to contact someone from the RH internal "unified messagebus" team,
search for "unified messagebus" in mojo.  It is operated as a joint project
between RHIT and PnT Devops.  See also the ``#devops-message`` IRC channel,
internally.
