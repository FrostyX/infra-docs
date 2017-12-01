.. title: simple-koji-ci SOP
.. slug: infra-simple-koji-ci
.. date: 2017-12-01
.. taxonomy: Contributors/Infrastructure

==============
simple-koji-ci
==============

simple-koji-ci is a small service running in our infra cloud that listens
for fedmsg messages coming from pagure on dist-git about new pull-requests.
It then creates a SRPM based on the content of each pull-request, kicks off
a scratch build in koji and reports the outcome of that build on the
pull-request.


Contact Information
===================

Owner
    Fedora Infrastructure Team
Contact
    #fedora-admin, #fedora-apps
Persons
    pingou
Location
    the cloud ☁
Servers
    simple-koji-ci-dev.fedorainfracloud.org
    simple-koji-ci-prod.fedorainfracloud.org
Purpose
    Performs scratch builds for pull-request opened on dist-git

Hosts
=====
The current deployment is made in a single host: ``simple-koji-ci-prod.fedorainfracloud.org``
for prod, ``simple-koji-ci-dev.fedorainfracloud.org`` for stagging.


Service
=======

simple-koji-ci is a fedmsg-based service, so it can be turned on or off via
the ``fedmsg-hub`` service.

It interacts with koji via a keytab created by the ``keytab/service`` role
in ansible.

The configuration of the service (including the weight of the builds kicked
off in koji) is located at ``/etc/fedmsg.d/simple_koji_ci.py``.

One can monitor the service using: ``journalctl -lfu fedmsg-hub``.


Impact
======

This service is purely informative, nothing does nor should rely on it. If
anything goes wrong, there are no consequences for stopping it.
