.. title: Zodbot Infrastucture SOP
.. slug: infra-zodbot
.. date: 2014-12-18
.. taxonomy: Contributors/Infrastructure

=========================
Zodbot Infrastructure SOP
=========================

zodbot is a supybot based irc bot that we use in our #fedora channels.

Contents
========

1. Contact Information
2. Description
3. shutdown
4. startup
5. Processing interrupted meeting logs
6. Becoming an admin

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	 Phoenix
Servers
	 value01
Purpose
	 Provides our IRC bot

Description
===========

zodbot is a supybot based irc bot that we use in our #fedora channels.
It runs on value01 as the daemon user. We do not config manage the
zodbot.conf because supybot makes changes to it on its own. Therefore it
gets backed up and is treated as data.

shutdown
  ``killall supybot``

startup
  `` cd /srv/web/meetbot`` 
  # zodbot current needs to be started in the meetbot directory.
  # This requirement will go away in a later meetbot release.
  ``sudo -u daemon supybot -d /var/lib/zodbot/conf/zodbot.conf``

Startup issues
==============

If the bot won't connect, with an error like::

  "Nick/channel is temporarily unavailable"

found in ``/var/lib/zodbot/logs/messages.log``, hop on Freenode (with your own
IRC client) and do the following::
  
  /msg nickserv release zodbot [the password]

The password can be found on the bot's host in
``/var/lib/zodbot/conf/zodbot.conf``

This should allow the bot to connect again.

Processing interrupted meeting logs
===================================

zodbot forgets about meetings if they are in progress when the bot goes
down; therefore, the meetings never get processed. Users may request a
ticket in [52]our Trac instance to have meeting logs processed.

Trac tickets for meeting log processing should consist of a URL where
zodbot had saved the log so far and an uploaded file containing the rest
of the log. The logs are stored in /srv/web/meetbot. Append the remainder
of the log uploaded to Trac (don't worry too much about formatting;
meeting.py works well with irssi- and XChat-like logs), then run::

  sudo python /usr/lib/python2.7/site-packages/supybot/plugins/MeetBot/meeting.py replay /path/to/fixed.log.txt

Close the Trac ticket, letting the user know that the logs are processed
in the same directory as the URL they gave you.

Becoming an admin
=================

Register with zodbot on IRC.::

  /msg zodbot misc help register

You have to identify to the bot to do any admin type commands, and you
need to have done so before anyone can give you privs.

After doing this, ask in #fedora-admin on IRC and someone will grant you
privs if you need them. You'll likely be added to the admin group, which
has the following capabilities (the below snippet is from an IRC log
illustrating how to get the list of capabilities).

::

  21:57 < nirik> .list admin
  21:57 < zodbot> nirik: capability add, capability remove, channels, ignore add,
  ignore list, ignore remove, join, nick, and part

