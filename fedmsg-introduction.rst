.. title: fedmsg Intro SOP
.. slug: infra-fedmsg-intro
.. date: 2012-10-31
.. taxonomy: Contributors/Infrastructure

===================================
fedmsg introduction and basics, SOP
===================================

General information about fedmsg

Contact Information
-------------------

Owner
	Messaging SIG, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-admin, #fedora-noc
Servers
	Almost all of them.
Purpose
	Introduce sysadmins to fedmsg tools and config

Description
-----------

fedmsg is a system that links together most of our webapps and services into
a message mesh or net (often called a "bus").  It is built on top of the
zeromq messaging library.

fedmsg has its own developer documentation that is a good place to check if
this or other SOPs don't provide enough information - http://fedmsg.rtfd.org

Tools
-----

Generally, fedmsg-tail and fedmsg-logger are the two most commonly used
tools for debugging and testing.  To see if bus-connectivity exists between
two machines, log onto each of them and run the following on the first::

  $ echo testing from $(hostname) | fedmsg-logger

And run the following on the second::

  $ fedmsg-tail --really-pretty

Configuration
-------------

fedmsg configuration lives in /etc/fedmsg.d/

``/etc/fedmsg.d/endpoints.py`` keeps the list of every possible fedmsg endpoint.
It acts as a global index that defines the bus.

See fedmsg.readthedocs.org/en/latest/config/ for a full glossary of
configuration values.

Logs
----

fedmsg daemons keep their logs in /var/log/fedmsg.  fedmsg message hooks in
existing apps (like bodhi) will log any errors to the logs of the app
they've been added to (like /var/log/httpd/error_log).
