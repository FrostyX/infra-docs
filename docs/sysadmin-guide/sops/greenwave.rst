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
	 dcallagh, gnaponie (giulia), lholecek, ralph (threebean)

Location
	 Phoenix

Public addresses
  - https://greenwave-web-greenwave.app.os.fedoraproject.org/api/v1.0/version
  - https://greenwave-web-greenwave.app.os.fedoraproject.org/api/v1.0/policies
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

You can roll out configuration changes by changing the files in
`roles/openshift-apps/greenwave/` and running the
`playbooks/openshift-apps/greenwave.yml` playbook.

To understand how the software is deployed, take a look at these two files:

- `roles/openshift-apps/greenwave/templates/imagestream.yml`
- `roles/openshift-apps/greenwave/templates/buildconfig.yml`

See that we build a fedora-infra specific image on top of an app image
published by upstream.  The `latest` tag is automatically deployed to
staging.  This should represent the latest commit to the `master` branch
of the upstream git repo that passed its unit and functional tests.

The `prod` tag is manually controlled.  To upgrade prod to match what is
in stage, move the `prod` tag to point to the same image as the `latest`
tag.  Our buildconfig is configured to poll that tag, so a new os.fp.o
build and deployment should be automatically created.

You can watch the build and deployment with `oc` commands.

You can poll this URL to see what version is live at the moment:
https://greenwave-web-greenwave.app.os.fedoraproject.org/api/v1.0/version

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
