.. title: Outage Infrastructure SOP
.. slug: infra-outage
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

=========================
Outage Infrastructure SOP
=========================

What to do when there's an outage or when you're planning to take an
outage.

Contents
========

1. Contact Information
2. Users (No Access)

  1. Planned Outage

    1. Contacts

  2. Unplanned Outage

    1. Check first
    2. Reporting or participating in an outage

5. Infrastructure Members (Admin Access)

  1. Planned Outage

    1. Planning
    2. Preparations
    3. Outage
    4. Post outage cleanup

  2. Unplanned Outage

    1. Determine Severity
    2. First Steps
    3. Fix it
    4. Escalate
    5. The Resolution
    6. The Aftermath

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main group
Location
	 Anywhere
Servers
	 Any
Purpose
	 This SOP is generic for any outage
Emergency: 
  https://admin.fedoraproject.org/pager


Users (No Access)
=================

.. note::
   Don't have shell access? Doesn't matter. Stop by and stay in #fedora-admin
   if you have any expertise in what is going on, please assist. Random users
   have helped the team out countless numbers of times. Any time the team
   doesn't have to go to the docs to look up an answer is a time they can be
   spending fixing what's busted.

Planned Outage
--------------

If a planned outage comes at a terrible time, just let someone know. The
Infrastructure Team does its best to keep outages out of the way but if
there's a mass rebuild going on that we don't know about and we schedule a
koji outage, let someone know.

Contacts
````````
   
Pretty much all coordination occurs in #fedora-admin on irc.freenode.net.
Stop by there to watch more about what's going on. Just stay on topic.

Unplanned Outage
----------------

Check first
```````````

Think something is busted? Please check with others to see if they are
also having issues. This could even include checking on another computer.
When reporting an outage remember that the admins will typically drop
everything they are doing to check what the problem is. They won't be
happy to find out your cert has expired or you're using the wrong
username. Additionally, check the status dashboard
(http://status.fedoraproject.org) to verify that there is no previously
reported outage that may be causing and/or related to your issue.

Reporting or participating in an outage
```````````````````````````````````````

If you think you've found an outage, get as much information as you can
about it at a glance. Copy any errors you get to http://pastebin.ca/.
Use the following guidelines:

Don't be general.
  * BAD: "The wiki is acting slow"
  * Good: "Whenever I try to save https://fedoraproject.org/wiki/Infrastructure,
    I get a proxy error after 60 seconds"

Don't report an outage that's already been reported.
  * BAD: "/join #fedora-admin; Is the build system broken?"
  * Good: "/join #fedora-admin; wait a minute or two; I noticed I
    can't submit builds, here's the error I get:"

Don't suggest drastic or needless changes during an outage (send it to the list)
  * "Why don't you just use lighttpd?"
  * "You could try limiting MaxRequestsPerChild in Apache"

Don't get off topic or be too chatty
  * "Transformers was awesome, but yeah, I think you guys know what to do next"

Do research the technologies we're using and answer questions that may come up.
  * BAD: "Can't you just fix it?"
  * Good: "Hey guys, I think this is what you're looking for:
      http://httpd.apache.org/docs/2.2/mod/mod_mime.html#addencoding"

If no one can be contacted after 10 minutes or so please see the section
below called Determine Severity to determine whether or not someone
should get paged.


Infrastructure Members (Admin Access) 
=====================================

The Infrastructure Members section is specifically written for members
with access to the servers. This could be admin access to a box or even a
specific web application. Basically anyone with access to fix the problem.

Planned Outage
--------------

Any outage that is intentionally caused by a team member is a planned
outage. Even if it has to happen in the next 5 minutes.

Planning
`````````

All major planned outages should occur with at least 1 week notice. This
is not always possible, use best judgment. Please use our standard outage
template at: https://fedoraproject.org/wiki/Infrastructure/OutageTemplate.
Make sure to have another person review your template/announcement to
check times and services affected. Make sure to send the announcement to
the lists that are affected by the outage: announce, devel-announce, etc.

Always create a ticket in the ticketing system:
https://fedoraproject.org/wiki/Infrastructure/Tickets
Send an email to the fedora-infrastructure-list with more details if
warranted.

Remember to follow an existing SOP as much as possible. If anything is
missing from the SOP please add it.

Preparations
`````````````

Remember to schedule an outage in nagios. This is important not just so
notifications don't get sent but also important for trending and
reporting. https://admin.fedoraproject.org/nagios/

Outage
``````

Prior to beginning an outage to any monitored service on
http://status.fedoraproject.org please push an update to reflect the outage
(see status-fedora SOP).

Report all information in #fedora-admin. Coordination is extremely
important, it's rare for our group to meet in person and IRC is our only
real-time communication device. If a web site is out please put up some
sort of outage page in its place.

Post outage cleanup
````````````````````

Once the outage is over ensure that all services are up and running.
Ensure all nagios services are back to green. Notify everyone in
#fedora-admin to scan our services for issues. Once all services are
cleared update the status.fp.o dashboard. If the outage included a
new feature or major change for a group, please notify that group that the
change is ready. Make sure to close the ticket for the outage when it's
over. 

Once the services are restored, an update to the status dashboard should be
pushed to show the services are restored.

.. important::
  Additionally update any SOP's that may have changed in the course of the
  outage

Unplanned Outage
----------------
Unplanned outages happen, stay cool. As a team member never be afraid to
do something because you think you'll get in trouble over it. Be smart,
don't be reckless, and never say "I shouldn't do this". If an unorthodox
method or drastic change will fix the problem, do it, document it, and let
the team know. Messes can always be cleaned up after the outage.

Determine Severity
``````````````````

Some outages require immediate fixing, some don't. A page should never go
out because someone can't sign the cla. Most of our admins are in US time,
use your best judgment. If it's bad enough to warrant an emergency page,
page one of the admins at: https://admin.fedoraproject.org/pager

Use the following as loose guidelines, just use your best judgment.

* BAD: "I can't see the Recent Changes on the wiki."
* Good: "The entire wiki is not viewable"

* BAD: I cannot sign the CLA
* Good: I can't change my password in the account system,
  I have admin access and my laptop was just stolen

* BAD: I can't access awstats for fedoraproject.org
* Good: The mirrors list is down.

* BAD: I think someone misspelled some words on the webpage
* Good: The web page has been hacked and I think someone
  notified slashdot.

First Steps
```````````

After an outage has been verified, acknowledge the outage in nagios:
https://admin.fedoraproject.org/nagios/, update the related system on the
status dashboard (see the status-fedora SOP) and verify changes at
http://status.fedoraproject.org, then head in to #fedora-admin
to figure out who is around and coordinate the next course of action.
Consult any relevent SOP's for corrective actions.

Fix it
```````
Fix it, Fix it, Fix it! Do whatever needs to be done to fix the problem,
just don't be stupid about it.

Escalate
`````````
Can't fix it? Don't wait, Escalate! All of the team members have expertise
with some areas of our environment and weaknesses in other areas. Never be
afraid to tap another team member. Sometimes it's required, sometimes it's
not. The last layer of defense is to page someone. At present our team is
small enough that a full escalation path wouldn't do much good. Consult
the contact information on each SOP for more information.

The Resolution
```````````````
Once the services are restored, an update to the status dashboard should be
pushed to show the services are restored. 

The Aftermath
``````````````
With any outage there will be questions. Please try as hard as possible to
answer the following questions and send them to the
fedora-infrastructure-list.

1. What happened?
2. What was affected?
3. How long was the outage?
4. What was the root cause?

.. important:: Number 4 is especially important. If a kernel build keeps failing because
  of issues with koji caused by a database failure caused by a full
  filesystem on db1. Don't say koji died because of a db failure. Any time a
  root cause is discovered and not being monitored by nagios, add it if
  possible. Most failures can be prevented or mitigated with proper
  monitoring.

