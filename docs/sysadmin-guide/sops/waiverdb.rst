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
	 mjia, dcallagh, cqi, qwan, sochotni, threebean

Location
	 Phoenix

Public addresses
  - https://waiverdb-web-waiverdb.app.os.fedoraproject.org/api/v1.0/waivers/

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

This is changing a lot right now, but look at the buildconfigs in
`roles/openshift-apps/waiverdb/files/imagestream.yml`.  That will show where
the container comes from.  It should be coming from the fedoraproject.org
candidate registry, built by OSBS.
