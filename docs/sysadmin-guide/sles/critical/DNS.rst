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
| Service       |  DNS                                   |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:   Stephen Smoogen            |
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
This document is limited to the DNS servers that the Fedora Project
provides as master domains for the various 'Fedora' owned domains.


Description of the Service:
===========================
The DNS nameservers are BIND named servers which provide data to the
Internet for all official FedoraProject domains (fedoraproject.org,
fedorapeople.org, fedoraplanet.org, etc). This allows for computers
connected to the Internet to be able to locate the IP addresses for
services attached to that. 

https://en.wikipedia.org/wiki/Domain_Name_System

Location of the Service:
========================
The Fedoraproject nameservers are split around the world at different
datacenters. The main servers are at PHX2 with some servers in Europe,
RDU2, and elsewhere.

Service functionality:
======================
The service allows for systems connected to the internet to do ip and
name lookups of systems controlled by the Fedora Project. It is
required to be working for most network related services to work. 

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

