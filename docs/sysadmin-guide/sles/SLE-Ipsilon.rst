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
| Service       |  IPSILON                               |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:   Patrick Uiterwijk          |
|               |  Secondary: Kevin Fenzi                |
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
This document is limited to the IPSILON service which backs up the
Fedora Account System.


Description of the Service:
===========================
IPSILON is our central authentication service which authenticates
users against FAS. The service is important to 2factor authentication
needed by system administration to sudo.

Location of the Service:
========================
The IPSILON services are are the PHX2 colocation. Client systems
connect to it through a web interface which is balanced by haproxy. 

Service functionality:
======================
The service is meant to authenticate users for web applications and
other systems.

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
This system should only allow logins by people in restricted sysadmin
groups. These groups will be regularly audited and admins removed as
needed. 

User Feedback Mechanism:
========================
Feedback on the service should be either given to the infrastructure
mailing list or the infrastructure ticketing system.

Service Reporting and Metrics:
==============================
- Monitoring: Nagios
- Reporting:
- Metrics:

Training and Documentation:
===========================
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/ipsilon.html

Cost:
=====
The services run on Dell r630's that are housed in the PHX2
location. Costs for this are covered by Red Hat Inc.

Development costs are covered by Fedora Infrastructure and volunteers.

Glossary of Terms:
==================

References:
===========
