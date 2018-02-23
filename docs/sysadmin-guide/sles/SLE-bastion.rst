=========
 Service
=========

+---------------+----------------------------------------+
| Field         |                                        |
+===============+========================================+
| Intial Date   |  2018-01-19                            |
+---------------+----------------------------------------+
| Last Updated  |  2018-01-19                            |
+---------------+----------------------------------------+
| Service       |  SSH Bastion                           |
|               |                                        |
+---------------+----------------------------------------+
| Service Owner |  Primary: Patrick Uiterwijk            |
|               |  Secondary: Stephen Smoogen            |
+---------------+----------------------------------------+
| Customer      |  Infrastructure, Release Engineering,  |
|               |  Quality Assurance                     |
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
This document is limited to the SSH on various bastion servers run by
Fedora Infrastructure.

Description of the Service:
===========================
The SSH Bastions are secure shell gateways from the Internet to Fedoraproject
internal networks. This is to allow various groups to be able to come
from their external networks using a cryptographic protocol. The main
purpose of the gateways are for interactive remote logins, however
they are also used as a throughpoint for other servers.

https://en.wikipedia.org/wiki/Secure_Shell

Location of the Service:
========================
The SSH bastions reside in two physical locations: the PHX2 and RDU2
Red Hat IT cages. The primary servers are in PHX2 and for disaester
recovery the ones in RDU2 are available. Secondary services are
available for Quality Assurance to get into their dedicated network.

Service functionality:
======================
The service needs to allow authorized users access to servers inside
of the associated facility. They should also allow for the user to
'hop' via appropriate methods to other servers they are authorized to
do so. 

Service Hours:
==============
This service is considered critical to Fedora and is to be up 24x7x365.


Service Availability:
=====================
The service at PHX2 should be available 99.9% of the time with a
backup server that users can switch to in case the primary is down. 

Incidents, Requests and Problem Management:
===========================================
Problems with bastion should be reported to Fedora Infrastructure
in the following order:
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
The authorized users of this server are required to do the following:
1. Keep the SSH keys they use to login password encrypted.
2. To not upload the private keys to non-private servers (cloud,
   bastion itself, fedorapeople, school home directory, etc.)
3. To report lost or stolen keys as quickly as found so that
   Infrastructure can audit systems.

The server will be routinely audited and the list of authorized users
will be regularly 'cleaned' of users who have not logged in within the
last 6 months.

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
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/bastion-hosts-info.html
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/sshaccess.html

Cost:
=====
The services run on Dell r630's that are housed in the PHX2
location. Costs for this are covered by Red Hat Inc.


Glossary of Terms:
==================

References:
===========

