=========
 Service
=========

+---------------+----------------------------------------+
| Field         |                                        |
+===============+========================================+
| Intial Date   |  2018-03-08                            |
+---------------+----------------------------------------+
| Last Updated  |  2018-03-08                            |
+---------------+----------------------------------------+
| Service       |  Infrastructure Git                    |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:   Kevin Fenzi                |
|               |  Secondary:                            |
+---------------+----------------------------------------+
| Customer      |  Infrastructure,                       |
|               |                                        |
+---------------+----------------------------------------+
| Priority      |  Critical                              |
+---------------+----------------------------------------+
| Availability  |  24x7x365                              |
+---------------+----------------------------------------+


=========================================
 Service Level Requirements/Expectations
=========================================

Scope:
======
This document is limited to the Fedora Infrastructure GIT repositories
which are maintained on a central server. These are used to hold all
configuration files for things like ansible, DNS, private passwords,
and other files which should be under version control.


Description of the Service:
===========================
This covers git file repositories

Location of the Service:
========================
These files are currently maintained on the batcave servers.


Service functionality:
======================
This is less of a service than a needed resource. All files which
should be considered centrally configured and controlled should be
kept in a GIT repository on the infrastructure server.


Service Hours:
==============
This service is considered critical to Fedora and is to be up 24x7x365.


Service Availability:
=====================
The service at PHX2 should be available 99.9% of the time with a
backup server that users can switch to in case the primary is down. 

Incidents, Requests and Problem Management:
=========================================== 
Problems should be reported to Fedora Infrastructure in the following
order:

* https://webchat.freenode.net/?channels=#fedora-admin
* https://pagure.io/fedora-infrastructure/issues
* https://admin.fedoraproject.org/pager


Maintenance Windows:
====================
Maintenance of the service will be announced through emails to the
following Fedora Project mailing lists:
https://lists.fedoraproject.org/archives/list/devel-announce@lists.fedoraproject.org/
https://lists.fedoraproject.org/archives/list/infrastructure@lists.fedoraproject.org/

Service Provider Responsibilities:
==================================
* Updating the system to latest security release
* Keeping the system backed up

Security and Governance:
========================
This resource should have write access limited to approved system
administrators. Read access for most repositories should be considered
public by any sysadmin. Files which are needing to be protected should
be kept in the private repository.

User Feedback Mechanism:
========================
Feedback on the service should be either given to the infrastructure
mailing list or the infrastructure ticketing system.

Service Reporting and Metrics:
==============================
- Monitoring: 
- Reporting:
- Metrics:

Training and Documentation:
===========================
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/infra-git-repo.rst

Cost:
=====
The services run on Dell r630's that are housed in the PHX2
location. Costs for this are covered by Red Hat Inc.

Glossary of Terms:
==================

References:
===========

