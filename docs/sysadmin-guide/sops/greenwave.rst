.. title: Greenwave SOP
.. slug: infra-greenwave
.. date: 2017-10-06
.. taxonomy: Contributors/Infrastructure

=============
Greenwave SOP
=============

Contact Information
===================

Owner
	 Factory2 Team, Fedora QA Team, Infrastructure Team

Contact
	 #fedora-qa, #fedora-admin

Persons
	 mjia, dcallagh, cqi, qwan, sochotni, threebean, gnaponie

Location
	 Phoenix

Public addresses
  - https://greenwave-web-greenwave.app.os.fedoraproject.org/api/v1.0/decision

Servers
  - In OpenShift.

Purpose
	Provide gating decisions.

Description
===========

- See `the focus document <http://fedoraproject.org/wiki/Infrastructure/Factory2/Focus/Greenwave>`_ for background.
- See `the upstream docs <https://pagure.io/docs/greenwave/>`_ for more detailed info.

Greenwave's job is:

- answering yes/no questions (or making decisions)
- about artifacts (RPM packages, source tarballs, …)
- at certain gating points in our pipeline
- based on test results
- according to some policy

In particular, we'll be using Greenwave to provide yes/no gating decisions *to
Bodhi* about rpms in each update.  Greenwave will do this by consulting
resultsdb and waiverdb for individual test results and then combining those
results into an aggregate decision.

The *policies* for how those results should be combined or ignored, are defined
in ansible in `roles/openshift-apps/greenwave/templates/configmap.yml`.
We expect to grow these over time to new use cases (rawhide compose gating, etc..)


Observing Greenwave Behavior
============================

Login to `os-master01.phx2.fedoraproject.org` as `root` (or, authenticate
remotely with openshift using `oc login https://os.fedoraproject.org`), and
run::

    $ oc project greenwave
    $ oc status -v
    $ oc logs -f dc/greenwave-web

Database
========

Greenwave currently has no database (and we'd like to keep it that way).  It
relies on `resultsdb` and `waiverdb` for information.

Upgrading
=========

This is changing a lot right now, but look at the buildconfigs in
`roles/openshift-apps/greenwave/files/buildconfig.yml`.  That will show where
the container comes from.

- It is currently being built *in* os.fedoraproject.org, from a greenwave rpm.
- Someday we will move that to an official fedoraproject.org container, built
  by OSBS (like waiverdb does it).

To upgrade, get a fresh build of the rpm.  Make sure the buildconfig can find
it.  Then run the playbook to trigger a fresh build.

Troubleshooting
===============

In case of problems with greenwave messaging, check the logs of the container
dc/greenwave-fedmsg-consumers to see if the is something wrong::

    $ oc logs -f dc/greenwave-fedmsg-consumers

It is also possible to check if greenwave is actually publishing messages
looking at `this link <https://apps.fedoraproject.org/datagrepper/raw?category=greenwave&delta=127800&rows_per_page=1>`_
and checking the time of the last message.

In case of problems with greenwave webapp, check the logs of the container
dc/greenwave-web::

    $ oc logs -f dc/greenwave-web
