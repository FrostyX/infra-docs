.. title: Module Build Service Infra SOP
.. slug: infra-mbs
.. date: 2017-06-01
.. taxonomy: Contributors/Infrastructure

==============================
Module Build Service Infra SOP
==============================

.. note::
   The MBS is very new and changing rapidly.  We'll try to keep this up to date
   as best we can.

The MBS is a build orchestrator on top of Koji for "modules".

https://fedoraproject.org/wiki/Changes/ModuleBuildService

Contents
========

1. Contact Information
2. Description
3. Observing MBS Behavior

  1. The mbs-build command
  2. The releng repo

4. Logs
5. Upgrading
6. Things that could go wrong

  1. Overloading Koji

Contact Information
===================

Owner
	 Factory2 Team, Release Engineering Team, Infrastructure Team

Contact
	 #fedora-modularity, #fedora-admin, #fedora-releng

Persons
	 jkaluza, fivaldi, mprahl, threebean

Location
	 Phoenix

Public addresses
  - mbs.fedoraproject.org

Servers
  - mbs-frontend0[1-2].phx2.fedoraproject.org
  - mbs-backend01.phx2.fedoraproject.org

Purpose
	 Build modules for Fedora.

Description
===========

Users submit builds to mbs.fedoraproject.org referencing their modulemd file in
dist-git.  (In the future, users will not submit their own module builds.  The
`freshmaker` daemon (running in infrastructure) will watch for .spec file
changes and submit the relevant module builds to the MBS on behalf of users.)

The request to build a module is received by the MBS flask app running on the
mbs-frontend nodes.

Cursory validation of the submitted modulemd is performed on the frontend: are
the named packages valid?  Are their branches valid?  The MBS keeps a copy of
the modulemd and appends additional data describing which branches pointed to
which hashes at the time of submission.

A fedmsg from the frontend triggers the backend to start building the module.
First, tags and build/srpm-build groups are created.  Then, a
module-build-macros package is synthesized and submitted as an srpm build.  When
it is complete and available in the buildroot, the rest of the rpm builds are
submitted.

These are grouped and limited in two ways:

- First, there is a global NUM_CONSECUTIVE_BUILDS config option that controls
  how many koji builds the MBS is allowed to have open at any time.  It serves
  as a throttle.
- Second, a given module specify that it's components should have a certain
  "build order".  If there are 50 components, it may say that a first 25 of them
  are in one buildorder batch, and the second 25 are in another buildorder
  batch.  The first batch will be submitted and, when complete, tagged back into
  the buildroot.  Only after they are available will the second batch of 25
  begin.

When the last component is complete, the MBS backend marks the build as "done",
and then marks it again as "ready".  (There is currently no meaning to the
"ready" state beyond "done".. we reserved it for future CI interactions.)

Observing MBS Behavior
----------------------

The mbs-build command
~~~~~~~~~~~~~~~~~~~~~

The `fm-orchestrator repo <https://pagure.io/fm-orchestrator>`_ and the
`module-build-service` package provide an `mbs-build` command with a few
subcommands.  For general help::

    $ mbs-build --help

To generate a report of all currently active module builds::

    $ mbs-build overview
      ID  State    Submitted             Components    Owner    Module
    ----  -------  --------------------  ------------  -------  -----------------------------------
     570  build    2017-06-01T17:18:11Z  35/134        psabata  shared-userspace-f26-20170601141014
     569  build    2017-06-01T14:18:04Z  14/15         mkocka   mariadb-f26-20170601141728

To generate a report of an individual module build, given its ID::

    $ mbs-build info 569
    NVR                                             State     Koji Task
    ----------------------------------------------  --------  ------------------------------------------------------------
    libaio-0.3.110-7.module_414736cc                COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803741
                                                    BUILDING  https://koji.fedoraproject.org/koji/taskinfo?taskID=19804081
    libedit-3.1-17.20160618cvs.module_414736cc      COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803745
    compat-openssl10-1.0.2j-6.module_414736cc       COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803746
    policycoreutils-2.6-5.module_414736cc           COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803513
    selinux-policy-3.13.1-255.module_414736cc       COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803748
    systemtap-3.1-5.module_414736cc                 COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803742
    libcgroup-0.41-11.module_ea91dfb0               COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19685834
    net-tools-2.0-0.42.20160912git.module_414736cc  COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19804010
    time-1.7-52.module_414736cc                     COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803747
    desktop-file-utils-0.23-3.module_ea91dfb0       COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19685835
    libselinux-2.6-6.module_ea91dfb0                COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19685833
    module-build-macros-0.1-1.module_414736cc       COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803333
    checkpolicy-2.6-1.module_414736cc               COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19803514
    dbus-glib-0.108-2.module_ea91dfb0               COMPLETE  https://koji.fedoraproject.org/koji/taskinfo?taskID=19685836


To actively watch a module build in flight, given its ID::

    $ mbs-build watch 570
    Still building:
       libXrender https://koji.fedoraproject.org/koji/taskinfo?taskID=19804885
       libXdamage https://koji.fedoraproject.org/koji/taskinfo?taskID=19805153
    Failed:
       libXxf86vm https://koji.fedoraproject.org/koji/taskinfo?taskID=19804903

    Summary:
       2 components in the BUILDING state
       34 components in the COMPLETE state
       1 components in the FAILED state
       97 components in the undefined state
    psabata's build #570 of shared-userspace-f26 is in the "build" state

The releng repo
~~~~~~~~~~~~~~~

There are more tools located in the `scripts/mbs/` directory of the releng
repo:  https://pagure.io/releng/blob/master/f/scripts/mbs

Logs
====

The frontend logs are on mbs-frontend0[1-2] in ``/var/log/httpd/error_log``.

The backend logs are on mbs-backend01.  Look in the journal for the
`fedmsg-hub` service.

Upgrading
=========

The package in question is `module-build-service`.  Please use the
`playbooks/manual/upgrade/mbs.yml` playbook.

Things that could go wrong
==========================

Overloading koji
----------------

If koji is overloaded, it should be acceptable to *stop* the fedmsg-hub daemon
on mbs-backend01 at any time.

.. note:: As builds finish in koji, they will be *missed* by the backend.. but
   when it restarts it should find them in datagrepper.  If that fails as well,
   the mbs backend has a poller which should start up ~5 minutes after startup
   that checks koji for anything it may have missed, at which point it will
   resume functioning.

If koji continues to be overloaded after startup, try decreasing the
`NUM_CONSECUTIVE_BUILDS` option in the config file in
`roles/mbs/common/templates/`.
