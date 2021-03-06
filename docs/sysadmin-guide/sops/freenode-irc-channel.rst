.. title: Freenode IRC SOP
.. slug: infra-freenode
.. date: 2013-11-08
.. taxonomy: Contributors/Infrastructure

=======================================
Freenode IRC Channel Infrastructure SOP
=======================================

Fedora uses the freenode IRC network for it's IRC communications. If you
want to make a new Fedora Related IRC Channel, please follow the following
guidelines.

Contents
========
     
1. Contact Information
2. Is a new channel needed?
3. Adding new channel
4. Recovering/fixing an existing channel

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin
Location: 
  freenode
Servers: 
  none
Purpose: 
  Provides a channel for Fedora contributors to use.

Is a new channel needed?
========================

First you should see if one of the existing Fedora channels will meet your
needs. Adding a new channel can give you a less noisy place to focus on
something, but at the cost of less people being involved. If you
topic/area is development related, perhaps the main #fedora-devel channel
will meet your needs?

Adding new channel
==================

* Make sure the channel is in the #fedora-* namespace. This allows the
  Fedora Group Coordinator to make changes to it if needed.

* Found the channel. You do this by /join #channelname, then /msg
  chanserv register #channelname

* Setup GUARD mode. This allows ChanServ to be in the channel for easier
  management: ``/msg chanserv set #channel GUARD on``

* Add Some other Operators/Managers to the access list. This would allow
  them to manage the channel if you are asleep or absent.::

   /msg chanserv access #channel add NICK +ARfiorstv

You can see what the various flags mean at http://toxin.jottit.com/freenode_chanserv_commands#cs03

You may want to consider adding some or all of the folks in #fedora-ops
who manage other channels to help you with yours. You can see this list
with `/msg chanserv access #fedora-ops list``

* Set default modes. 
  ``/msg chanserv set mlock #channel +Ccnt``
  (The t for topic lock is optional, if your channel would like 
  to have people change the topic often).

* If your channel is of general interest, add it to the main communicate
  page of IRC Channels, and possibly announce it to your target
  audience.

* You may want to request zodbot join your channel if you need it's
  functions. You can request that in #fedora-admin.

Recovering/fixing an existing channel
=====================================

If there is an existing channel in the #fedora-* namespace that has a
missing founder/operator, please contact the Fedora Group Coordinator:
[49]User:Spot and request it be reassigned. Follow the above procedure
on the channel once done so it's setup and has enough
operators/managers to not need reassiging again.

