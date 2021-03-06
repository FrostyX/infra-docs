.. title: fedmsg IRC SOP
.. slug: infra-fedmsg-irc
.. date: 2014-02-13
.. taxonomy: Contributors/Infrastructure

==============
fedmsg-irc SOP
==============

  Echo fedmsg bus activity to IRC.

Contact Information
===================

Owner
	Messaging SIG, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-fedmsg, #fedora-admin, #fedora-noc
Servers
	value03
Purpose
	Echo fedmsg bus activity to IRC

Description
===========

fedmsg-irc is a daemon running on value03 and value01.stg.  It is listening
to the fedmsg bus and echoing that activity to the #fedora-fedmsg channel in
IRC.

It can be configured to ignore certain messages, join certain rooms, and
take on a different nick by editing the values in ``/etc/fedmsg.d/irc.py`` and
restarting it with ``sudo service fedmsg-irc restart``

See http://fedmsg.readthedocs.org/en/latest/config/#term-irc for more
information on configuration.
