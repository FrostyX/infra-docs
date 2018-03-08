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
| Service       |  Fedora Account System                 |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:   Patrick Uiterwijk          |
|               |  Secondary: Kevin Fenzi                |
+---------------+----------------------------------------+
| Customer      |  Infrastructure, All Fedora Account    |
|               |  Users                                 |
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
This document is limited to the Fedora Account System (FAS) which is
an application written by Fedora Infrastructure for authentication,
and authorization of users, groups and services within the Fedora
Project infrastructure. 


Description of the Service:
===========================

The Fedora Account System was written to cover the account needs of
the Fedora Project. It ties into every part of the Fedora Project
ranging from who can commit to packages, who can build packages, who
can log into servers and what rules bots and services may have in
interacting with users. The system was originally written as a web
application which used SSL certificates and custom passwd databases on
each client. It is now backed in part by another service of kerberos
and IPSILON.

Location of the Service:
========================
The main FAS servers are in the PHX2 colocation. All client systems
will update their logins by querying the servers and the FAS servers
will then talk to a backing postgres database.

Service functionality:
======================
This service allows for users to log into systems and operate nearly
all Fedora services.

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
- Reporting:
- Metrics:

Training and Documentation:
===========================
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/fas-notes.rst
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/fas-openid.rst


Cost:
=====
The virtual systems run on Dell r630's that are houses in the PHX2
colocation. Costs for this are covered by Red Hat.

Glossary of Terms:
==================

References:
===========

