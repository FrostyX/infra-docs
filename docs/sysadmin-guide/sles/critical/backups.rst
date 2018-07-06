=========
 Service
=========

+---------------+----------------------------------------+
| Field         |                                        |
+===============+========================================+
| Intial Date   |  2018-04-09                            |
+---------------+----------------------------------------+
| Last Updated  |  2018-04-09                            |
+---------------+----------------------------------------+
| Service       |  Backups                               |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:   Kevin Fenzi                |
|               |  Secondary:                            |
+---------------+----------------------------------------+
| Customer      |  Infrastructure, All Fedora Users      |
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
This document is limited to the backups system that 


Description of the Service:
===========================

Location of the Service:
========================

Service functionality:
======================

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

User Feedback Mechanism:
========================
Feedback on the service should be either given to the infrastructure
mailing list or the infrastructure ticketing system.

Service Reporting and Metrics:
==============================
- Monitoring: Nagios
- Reporting:  Collectd
- Metrics:

Training and Documentation:
===========================
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/dns.html


Cost:
=====
Several virtual systems run on Dell r630's that are housed in the PHX2
location. Costs for this are covered by Red Hat Inc.

Other virtual systems run on servers provided by various ISPs. Network
costs are covered by them.


Glossary of Terms:
==================

References:
===========

