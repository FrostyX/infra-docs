.. title: Basset antispam documentation
.. slug: infra-basset
.. date: 2016-03-18
.. taxonomy: Contributors/Infrastructure

========================
Basset anti-spam service
========================

Since the Fedora Project has come under targeted spam attacks, we
have decided to create a service that all our applications can hook
into to have a central repository for anti-spam procedures.
Basset is this service, and it's hosted on https://pagure.io/basset.


Contents
========
  
1. Contact Information
2. Overview
3. FAS
4. Trac
5. Wiki
6. Setup
7. Outage


Contact Information
===================

Owner
  Patrick Uiterwijk (puiterwijk)
Contact
  #fedora-admin, #fedora-apps, #fedora-noc, sysadmin-main
Location
  basset01
Purpose
  Centralized anti-spam


Overview
========

Basset is a central anti-spam service: it received messages from services that
certain actions happened, and will then decide to accept or deny the request, or
pass it on to an administrator.

At the moment, we have the following modules live: FAS, trac, wiki.

FAS
===
This module receives notifications from FAS about new users registrations and new
users signing the FPCA.
With Basset enabled, FAS will not automatically accept a new user registration or
a FPCA signing, but instead let Basset know a user tried to perform these actions
and then depend on Basset to enact this.

In the case of registration this is done by setting the user to a spamcheck_awaiting
status. As soon as Basset made a decision, it will set the user to spamcheck_manual,
spamcheck_denied or active.
If it sets the user to active, it will also send the welcome email to the user.
If it made a wrong decision, and the user is set as spamcheck_manual or spamcheck_denied,
a member of the accounts team can go to that users' page and click the "Enable" button
to override the decision.
If this needed to be done, please notify puiterwijk so that the rules Basset uses
can be updated.

For the case of the FPCA, FAS will request the cla_fpca group membership,
but not sponsor the user. At the moment that Basset decides it accepts the request,
it will sponsor the user into the group.
If it declined the FPCA request, it will remove the user from the group.
To override this decision, a member of the accounts group can go to FAS and manually
add the user to the cla_fpca group and sponsor them into it.


Trac
====

For Trac, if a post gets denied, the content item gets deleted, the Trac account gets
blocked cross-instance and the FAS account gets blocked.

To unblock the user, log in to hosted03, and remove /srv/web/trac/blocks/$username.
For info on how to unblock the FAS user, see the notes under FAS.


Wiki
====

For Wiki, if an edit gets denied, the page gets deleted, the wiki account blocked and the
FAS account gets blocked.

For the wiki parts of undoing this, follow the regular mediawiki unblock procedures using:
 - https://fedoraproject.org/wiki/Special:BlockList to check if an user is blocked or not
 - https://fedoraproject.org/wiki/Special:Unblock to unblock that user

Don't forget to unblock the account as in FAS.


Setup
=====

At this moment, Basset runs on a single server (basset01(.stg)), and runs the frontend,
message broker and worker all on a single server.
For all of it to work, the following services are used:
- httpd (frontend)
- rabbitmq-server (broker)
- mongod (mongo database server for storage of internal info)
- basset-worker (worker)


Outage
======

The consequences of certain services not being up results in various conditions:

If the httpd or frontend aren't up, no new messages will come in.
FAS will set the user to spamcheck_awaiting, but not submit it to Basset.
Work is in progress on a script to submit such entries to the queue after Basset frontend
is back.
However, since this part of the code is so small, this is not likely to be the part that's down.
(You can know that it is because the FAS logs will log an error instead of "result: checking".)

If the worker or the mongo server are down, no messages will be processed, but all messages
queued up will be processed the moment both of the services start again: as long as a message
makes it into the queue, it will be processed until completion.

If the worker encounters an error during processing of a message, it will dump a tracedump
into the journal log file, and stop processing any messages.
Resolve the condition reported in the error and restart the basset-worker service, and all
work will be continued, starting with the message it was processing when it errored out.

This means that as long as the message is queued, the worker will pick it up and handle it.
