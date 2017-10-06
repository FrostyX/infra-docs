.. title: Infrastructure Request for Resources SOP
.. slug: infra-rfr
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

=========================
Request for resources SOP
=========================

Contents
=========

1. Contact Information
2. Introduction
3. Pre sponsorship
4. Planning
5. Development Instance
6. Staging Instance
7. Production deployment
8. Maintenance

Contact Information
====================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin
Location
 fedoraproject.org/wiki
Servers
 dev, stg, production
Purpose
 Explains the technical part of Request for Resources

Introduction
============

Once a RFR has a sponsor and has been generally agreed to move forward,
this SOP will describe the technical parts of moving a RFR through the
various steps it needs from idea to implementation. Note that for high
level and non technical requirements, please see the main RFR page.

A RFR will go through (at least) the following steps, but note that it can
be dropped, removed or reverted at any time in the process and that MUST
items MUST be provided before the next step is possible.

Pre sponsorship
===============

Until a RFR has a sysadmin-main person who is sponsoring and helping with
the request, no further technical action should take place with this SOP.
Please see the main RFR SOP to aquire a sponsor and do the steps needed
before implementation starts. If your resource requires packages to be
complete, please finish your packaging work before moving forward with the
RFR (accepted/approved packages in Fedora/EPEL). If your RFR only has a
single person working on it, please gather at least another person before
moving forward. Single points of failure are to be avoided.

Requirements for continuing:
----------------------------

* MUST have a RFR ticket.

* MUST have the ticket assigned and accepted by someone in
  infrastructure sysadmin-main group.

Ticket comment template:
------------------------

* **Software**: <mynewservice>
* **Advantage for Fedora**: <It will give us unicorns>
* **Sponsor**: <someone>


Planning
========

Once a sponsor is aquired and all needed packages have been packaged and
are available in EPEL, we move on to the planning phase. In this phase
discussion should take place about the application/resource on the
infrastructure list and IRC. Questions about how the resource could be
deployed should be considered:

* Should the resource be load balanced?

* Does the resource need caching?

* Can the resource live on it's own instance to separate it from more
  critical services?

* Who all is involved in maintaining and deploying the instance?

Requirements for continuing:
----------------------------

* MUST discuss/note the app on the infrastructure mailing list and
  answer feedback there.

* MUST determine who is involved in the deployment/maintaining the
  resource.

Ticket comment template:
------------------------

* **Email list thread**: <https://lists.fedoraproject.org/....>
* **Upstream source**: <https://github.com/...>
* **Development contacts**: <person1, person2>
* **Maintainership contacts**: <person2, person3>
* **Load balanceable**: <yes/no>
* **Caching**: <yes/no, which paths, ...>

Development Instance
====================

In this phase a development instance is setup for the resource. This
instance is a single virtual host running the needed OS. The RFR sponsor
will create this instance and also create a group 'sysadmin-resource' for
the resource, adding all responsible parties to the group. It's then up to
sysadmin-resource members to setup the resource and test it. Questions
asked in the planning phase should be investigated once the instance is
up. Load testing and other testing should be performed. Issues like
expiring old data, log files, acceptable content, packaging issues,
configuration, general bugs, security profile, and others should be
investigated. At the end of this step a email should be sent to the
infrastucture list explaining the testing done and inviting comment.
Also, the security officer should be informed that a new service will
need a review in the near future.

Requirements for continuing:
----------------------------

* MUST have RFR sponsor sign off that the resource is ready to move to
  the next step.

* MUST have answered any outstanding questions on the infrastructure
  list about the resource. Decisions about caching, load balancing and
  how the resource would be best deployed should be determined.

* MUST add any needed SOP's for the service. Should there be an Update
  SOP? A troubleshooting SOP? Any other tasks that might need to be done
  to the instance when those who know it well are not available?

* MUST tag in the security officer in the ticket so an audit can be scheduled.

Ticket comment template:
------------------------

* **SOP link**: <https://docs.pagure.io/infra-docs/.....>
* **Audit request**: <https://pagure.io/fedora-infrastructure/issue/....> (can be same)
* **Audit timeline**: <04-11-2025 - 06-11-2025>

Staging Instance
================

The next step is to create a staging instance for the resource. In this
step the resource is fully added to Ansible/configuration management. The
resource is added to caching/load balancing/databases and tested in this
new env. Once initial deployment is done and tested, another email to the
infrastructure list is done to note that the resource is available in
staging.

The security officer should be informed as soon as the code is reasonably
stable, so that they can start the audit or delegate the audit to someone.

Requirements for continuing:
----------------------------

* MUST have sign off of RFR sponsor that the resource is fully
  configured in Ansible and ready to be deployed.

* MUST have a deployment schedule for going to production. This will
  need to account for things like freezes and availability of
  infrastructure folks.

* MUST have an approved audit by the security officer or appointed delegate.

Ticket comment template:
------------------------

* **Ansible playbooks**: <ansible/playbooks/groups/myservice.yml>
* **Fully rebuilt from ansible**: <yes>
* **Production goal**: <08-11-2025>
* **Approved audit**: <https://pagure.io/fedora-infrastructure/issue/....>

Production deployment
=====================

Finally the staging changes are merged over to production and the resource
is deployed.

Monitoring of the resource is added and confirmed to be effective.

Maintenance
===========

The resource will then follow the normal rules for production. Honoring
freezes, updating for issues or security bugs, adjusting for capacity,
etc.

