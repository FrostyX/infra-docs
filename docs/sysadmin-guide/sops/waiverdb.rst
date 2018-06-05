.. title: WaiverDB SOP
.. slug: infra-waiverdb
.. date: 2017-10-06
.. taxonomy: Contributors/Infrastructure

============
WaiverDB SOP
============

WaiverDB is a service for recording waivers, from humans, that correspond with
results in ResultsDB.

On its own, this doesn't do much.

Importantly, the *Greenwave* service queries resultsdb *and* waiverdb and makes
decisions (for *Bodhi* and other tools) based on the combination of data from
the two sources.  A result in resultsdb may matter, unless waived in waiverdb.

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
  - https://waiverdb-web-waiverdb.app.os.fedoraproject.org/api/v1.0/about
  - https://waiverdb-web-waiverdb.app.os.fedoraproject.org/api/v1.0/waivers

Servers
  - In OpenShift.

Purpose
	Record waivers and respond to queries about them.

Description
===========

See the `the upstream API docs for detailed information
<https://pagure.io/docs/waiverdb/>`_.  The information here will be contextual
to the Fedora environment.

There *will be* two ways of inserting waivers into waiverdb:

First, a cli tool, which performs a HTTP POST from the packager's machine.

Second, a proxied request from bodhi.  In this case, the packager will click a
button in the Bodhi UI (next to a failing test result).  Bodhi will receive the
request from the user and in turn submit a POST to waiverdb on the user's
behalf.  Here, the Bodhi Server will authenticate *as* the bodhi user, but
request that the waiver be recorded as having been submitted *by* the original
packager.  Bodhi's account will have to be given special *proxy* privileges in waiverdb.
See https://pagure.io/waiverdb/issue/77

Observing WaiverDB Behavior
===========================

Login to `os-master01.phx2.fedoraproject.org` as `root` (or, authenticate
remotely with openshift using `oc login https://os.fedoraproject.org`), and
run::

    $ oc project waiverdb
    $ oc status -v
    $ oc logs -f dc/waiverdb-web

Removing erroneous waivers
==========================

In general, don't do this.  But if for some reason we *really* need to, the
database for waiverdb lives outside of openshift in our standard environment.
Connect to db01::

    [root@db01 ~][PROD]# sudo -u postgres psql waiverdb

    waiverdb=# \d
                  List of relations
     Schema |     Name      |   Type   |  Owner
    --------+---------------+----------+----------
     public | waiver        | table    | waiverdb
     public | waiver_id_seq | sequence | waiverdb
    (2 rows)

    waiverdb=# select * from waiver;

Be careful.  You can delete individual waivers with SQL.

Upgrading
=========

You can roll out configuration changes by changing the files in
`roles/openshift-apps/waiverdb/` and running the
`playbooks/openshift-apps/waiverdb.yml` playbook.

To understand how the software is deployed, take a look at these two files:

- `roles/openshift-apps/waiverdb/templates/imagestream.yml`
- `roles/openshift-apps/waiverdb/templates/buildconfig.yml`

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
https://waiverdb-web-waiverdb.app.os.fedoraproject.org/api/v1.0/about
