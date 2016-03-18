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

At the moment, only the FAS module is live for Fedora.
This module receives notifications from FAS about new users registrations and new
users signing the FPCA.
With Basset enabled, FAS will not automatically accept a new user registration or
a FPCA signing, but instead let Basset know a user tried to perform these actions
and then depend on Basset to enact this.

In the case of registration this is done by setting the user to a spamcheck_awaiting
status. As soon as Basset made a decision, it will set the user to spamcheck_manual,
spamcheck_denied or active.
If it sets the user to active, it will also send the welcome email to the user.
If if made a wrong decision, and the user is set as spamcheck_manual or spamcheck_denied,
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


Setup
=====

At this moment, Basset runs on a single server (basset01(.stg)), and runs the frontend,
message broker and worker all on a single server.
For all of it to work, the following services are used:
- httpd (frontend)
- rabbitmq-server (broker)
- mongod (mongo database server for storage of internal info)
- basset-worker (worker)
