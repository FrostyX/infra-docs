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
| Service       |  Ansible                               |
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
This document is limited to the Fedora Infrastructure ansible command
and repository. 


Description of the Service:
===========================
The ansible program is a deployment software which will operate
command by command over a set of configurations. The Fedora
Infrastructure uses a central configuration of ansible to control and
deploy most of the servers used.

Location of the Service:
========================
The central ansible server is deployed in the PHX2 colocation. A
secondary backup server is maintained offline in RDU2.

Service functionality:
======================
The service is a scripted installation service versus a full
configuration management system like puppet, chef, or cfengine. There
is no service running 24x7 on batcave which checks configurations and
updates clients. Instead after changes have been made, the system
administrator will run an ansible-playbook or script to finish client
configuration. 

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
The ansible playbooks can be run by only people who have certain
sysadmin classes or are able to run it via a rbac command system.

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
https://docs.pagure.org/infra-docs/sysadmin-guide/sops/ansible.rst

Cost:
=====
The services run on Dell r630's that are housed in the PHX2
location. Costs for this are covered by Red Hat Inc.


Glossary of Terms:
==================

References:
===========

