=========
 Service
=========

+---------------+----------------------------------------+
| Field         |                                        |
+===============+========================================+
| Initial Date  |  2018-01-19                            |
+---------------+----------------------------------------+
| Last Updated  |  2018-01-19                            |
+---------------+----------------------------------------+
| Service       |                                        |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary:                              |
|               |  Secondary:                            |
+---------------+----------------------------------------+
| Customer      |                                        |
|               |                                        |
+---------------+----------------------------------------+
| Priority      |  Deprecated                            |
+---------------+----------------------------------------+
| Availability  |  50 weeks x 5 days Regular Bus         |
+---------------+----------------------------------------+

=========================================
 Service Level Requirements/Expectations
=========================================

Scope:
======

This document covers the bittorrent services available on the internet
for users to download a restricted set of isos from.

Description of the Service:
===========================
Torrent acts as a seeder for various releases and spins so that users
may use the bittorrent protocol to download ISOs. Torrents were useful
for users in low bandwidth areas to get faster downloads however they
require a large number of volunteer participants.


Location of the Service:
========================
The torrent seeder resides in Ibiblio network.

Service functionality:
======================


Service Hours:
==============

This service is considered deprecated and to be end of lifed at some
point in the future. Repairs to the service will be below any higher
level service. While generally available 24x7x365 may be taken down
and fixed during Normal Business Hours.

Service Availability:
=====================
This service should be available for 90% per day.

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

Service Provider Responsibilities
=================================
* Updating the system to latest security release
* Keeping the system backed up

Security and Governance:
========================
The torrent seeder software and related software have had limited
support from their upstreams. The service may be blocked on many
networks as torrent sharing can cause audit problems.

User Feedback Mechanism:
========================
Service problems should have tickets opened in the Fedora
infrastructure ticketing system.

Service Reporting and Metrics:
==============================
- Monitoring:
- Reporting:
- Metrics:

Training and Documentation:
===========================
There is no training or documentation on this service.

Other Service Provider Responsibilities:
========================================
* Responding to outages and seeing if they can be fixed
* Cleaning up old torrents

Cost:
=====
This service has different costs to different groups. This tool is not
automated so releng has to build torrents each release. This tool
sometimes gets blocked on networks it is served from. The software
does only snapshots of usage so tracking how well it is serving the
community is hard.


Glossary of Terms:
==================

References:
===========

