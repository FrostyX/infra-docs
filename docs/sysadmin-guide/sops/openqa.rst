.. title: openQA Infrastructure SOP
.. slug: infra-openqa
.. date: 2018-01-10
.. taxonomy: Contributors/Infrastructure

=========================
openQA Infrastructure SOP
=========================

openQA is an automated test system used to run validation tests on nightly and candidate
Fedora composes, and also to run a subset of these tests on critical path updates.

openQA production instance: https://openqa.fedoraproject.org
openQA staging instance: https://openqa.stg.fedoraproject.org
Wiki page on Fedora openQA deployment: https://fedoraproject.org/wiki/OpenQA
Upstream project page: http://open.qa/
Upstream repositories: https://github.com/os-autoinst


Contact Information
===================

Owner
  Fedora QA devel

Contact
  #fedora-qa, #fedora-admin, qa-devel mailing list

People
  Adam Williamson (adamwill / adamw), Petr Schindler (pschindl)

Location
  PHX2

Machines
  See ansible inventory groups with 'openqa' in name

Purpose
  Run automated tests on VMs via screen recognition and VNC input


Architecture
============

Each openQA instance consists of a server (these are virtual machines) and one or more worker
hosts (these are bare metal systems). The server schedules tests ("jobs", in openQA parlance)
and stores results and associated data. The worker hosts run "jobs" and send the results back
to the server. The server also runs some fedmsg consumers to handle automatic scheduling of
jobs and reporting of results to external systems (ResultsDB and Wikitcms).


Server
======

The server runs a web UI for viewing scheduled, running and completed tests and their data, with
an admin interface where many aspects of the system can be configured (though we do not use the
web UI for several aspects of configuration). There are several separate services that run on each
server, and communicate with each other mainly via dbus. Each server requires its own PostgreSQL
database. The web UI and websockets server are made externally available via reverse proxying
through an Apache server.

It hosts an NFS share that contains the tests, the 'needles' (screenshots with metadata as JSON
files that are used for screen matching), and test 'assets' like ISO files and disk images. The
path is `/var/lib/openqa/share/factory`.

In our deployment, the PostgreSQL database for each instance is hosted by the QA database server.
Also, some paths on the server are themselves mounted as NFS shares from the infra storage server.
This is so that these are not lost if the server is re-deployed, and can easily be backed up.
These locations contain the data from each executed job. As both the database and these key
data files are not actually stored on the server, the server can be redeployed from scratch
without loss of any data (at least, this is the intent).

Also in our deployment, an openQA plugin (which we wrote, but which is part of the upstream
codebase) is enabled which emits fedmsgs on various events. This works by calling fedmsg-logger,
so the appropriate fedmsg configuration must be in place for this to emit events correctly.

The server systems run a fedmsg consumer for the purpose of automatically scheduling jobs in
response to the appearance of new composes and critical path updates, and one for the purpose
of reporting the results of completed jobs to ResultsDB and Wikitcms. These use the `fedmsg-hub`
system.


Worker hosts
============

The worker hosts run several individual worker 'instances' (via systemd's 'instantiated service'
mechanism), each of which registers with the server and accepts jobs from it, uploading the
results of the job and some associated data to the server on completion. The worker instances
and server communicate both via a conventional web API provided by the server and via websockets.
When a worker runs a job, it starts a qemu virtual machine (directly - libvirt is not used) and
interacts with it via VNC and the serial console, following a set of steps dictating what it
should do and what response it should expect in terms of screen contents or serial console
output. The server 'pushes' jobs to the worker instances over a websocket connection.

Each worker host must mount the `/var/lib/openqa/share/factory` NFS share provided by the server.
If this share is not mounted, any jobs run will fail immediately due to expected asset and test
files not being found.

Some worker hosts for each instance are denominated 'tap workers', meaning they run some advanced
jobs which use software-defined networking (openvswitch) to interact with each other. All the
configuration for this should be handled by the ansible scripts, but it's useful to be aware that
there is complex software-defined networking stuff going on on these hosts which could potentially
be the source of problems.


Deployment and regular operation
================================

Deployment and normal update of the openQA systems should run entirely through Ansible. Just
running the appropriate ansible plays for the systems should complete the entire deployment /
update process, though it is best to check after running them that there are no failed services
on any of the systems (restart any that failed), and that the web UI is properly accessible.

Regular operation of the openQA deployments is entirely automated. Jobs should be scheduled
and run automatically when new composes and critical path updates appear, and results should
be reported to ResultsDB and Wikitcms (when appropriate). Dynamically generated assets should
be regenerated regularly, including across release boundaries (see the section on createhdds
below): no manual intervention should be required when a new Fedora release appears. If any of
this does not happen, something is wrong, and manual inspection is needed.

Our usual practice is to upgrade the openQA systems to new Fedora releases promptly as they
appear, using `dnf system-upgrade`. This is done manually. We usually upgrade the staging instance
first and watch for problems for a week or two before upgrading production.


Rebooting / restarting
======================

The optimal approach to rebooting an entire openQA deployment is as follows:

1. Wait until no jobs are running
2. Stop all `openqa-*` services on the server, so no more will be queued
3. Stop all `openqa-worker@` services on the worker hosts
4. Reboot the server
5. Check for failed services (`systemctl --failed`) and restart any that failed
6. Once the server is fully functional, reboot the worker hosts
7. Check for failed services and restart any that failed, particularly the NFS mount service

Rebooting the workers **after** the server is important due to the NFS share.

If only the server needs restarting, the entire procedure above should ideally be followed
in any case, to ensure there are no issues with the NFS mount breaking due to the server reboot,
or the server and worker getting confused about running jobs due to the websockets connections
being restarted.

If only a worker host needs restarting, there is no need to restart the server too, but it is best
to wait until no jobs are running on that host, and stop all `open-worker@` services on the host
before rebooting it.

There are two ways to check if jobs are running and if so where. You can go to the web UI for
the server and click 'All Tests'. If any jobs are running, you can open each one individually
(click the link in the 'Test' column) and look at the 'Assigned worker', which will tell you
which host the job is running on. Or, if you have admin access, you can go to the admin menu
(top right of the web UI, once you are logged in) and click on 'Workers', which will show the
status of all known workers for that server, and select 'Working' in the state filter box.
This will show all workers currently working on a job.

Note that if something which would usually be tested (new compose, new critpath update...) appears
during the reboot window, it likely will *not* be scheduled for testing, as this is done by a
fedmsg consumer running on the server. You will need to schedule it for testing manually in this
case (see below).


Scheduling jobs manually
========================

While it is not normally necessary, you may sometimes need to run or re-run jobs manually.

The simplest cases can be handled by an admin from the web UI: for a logged-in admin, all scheduled
and running tests can be cancelled (from various views), and all completed tests can be restarted.
'Restarting' a job actually effectively clones it and schedules the clone to be run: it creates a
new job with a new job ID, and the previous job still exists. openQA attempts to handle complex
cases of inter-dependent jobs correctly when restarting, but doesn't always manage to do it right;
when it goes wrong, the best thing to do is usually to re-run all jobs for that medium.

To run or re-run the full set of tests for a compose or update, you can use the `fedora-openqa`
CLI. To run or re-run tests for a compose, use:

    fedora-openqa compose -f (COMPOSE LOCATION)

where (COMPOSE LOCATION) is the full URL of the /compose subdirectory of the compose. This will
only work for Pungi-produced composes with the expected productmd-format metadata, and a couple
of other quite special cases.

The `-f` argument means 'force', and is necessary to re-run tests: usually, the scheduler will
refuse to re-schedule tests that have already run, and `-f` overrides this.

To run or re-run tests for an update, use:

    fedora-openqa update -f (UPDATEID) (RELEASE)

where (UPDATEID) is the update's ID - something like `FEDORA-2018-blahblah` - and (RELEASE) is the
release for which the update is intended (27, 28, etc).

To run or re-run only the tests for a specific medium (usually a single image file), you must use
the lower-level web API client, with a more complex syntax. The command looks something like this:

    /usr/share/openqa/script/client isos post ISO=Fedora-Server-dvd-x86_64-Rawhide-20180108.n.0.iso DISTRI=fedora VERSION=Rawhide FLAVOR=Server-dvd-iso ARCH=x86_64 BUILD=Fedora-Rawhide-20180108.n.0 CURRREL=27 PREVREL=26 RAWREL=28 IMAGETYPE=dvd LOCATION=http://kojipkgs.fedoraproject.org/compose/rawhide/Fedora-Rawhide-20180108.n.0/compose SUBVARIANT=Server

The `ISO` value is the filename of the image to test (it may not actually be an ISO), the `DISTRI`
value is always 'fedora', the `VERSION` value should be the release number or 'Rawhide', the
`FLAVOR` value depends on the image being tested (you can check the value from an existing test
for the same or a similar ISO), the `ARCH` value is the arch of the image being tested, the `BUILD`
value is the compose ID, `CURREL` should be the release number of the current Fedora release at
the time the test is run, `PREVREL` should be one lower than `CURREL`, `RAWREL` should be the
release number associated with Rawhide at the time the test is run, `IMAGETYPE` depends on the
image being tested (again, check a similar test for the correct value), `LOCATION` is the URL to
the /compose subdirectory of the compose location, and `SUBVARIANT` again depends on the image
being tested. Please ask for help if this seems too daunting. To re-run the 'universal' tests on a
given image, set the `FLAVOR` value to 'universal', then set all other values as appropriate to the 
chosen image. The 'universal' tests are only likely to work at all correctly with DVD or netinst
images.

openQA provides a special script for cloning an existing job but optionally changing one or more
variable values, which can be useful in some situations. Using it looks like this:

    /usr/share/openqa/script/clone_job.pl --skip-download --from localhost 123 RAWREL=28

to clone job 123 with the RAWREL variable set to '28', for instance. For interdependent jobs, you
may or may not want to use the `--skip-deps` argument to avoid re-running the cloned job's parent
job(s), depending on circumstances.


Manual updates
==============

In general updates to any of the components of the deployments should be handled via ansible:
push the changes out in the appropriate way (git repo update, package update, etc.) and then run
the ansible plays. However, sometimes we do want to update or test a change to something manually
for some reason. Here are some notes on those cases.

For updating openQA and/or os-autoinst packages: ideally, ensure no jobs are running. Then, update
all installed subpackages on the server. The server services should be automatically restarted as
part of the package update. Then, update all installed subpackages on the worker hosts, and restart
all worker services. A 'for' loop can help with that, for instance:

    for i in 1 2 3 4 5 6 7 8 9 10; do systemctl restart openqa-worker@$i.service; done

on a host with ten worker instances.

For updating the openQA tests:

    cd /var/lib/openqa/share/tests/fedora
    git pull (or git checkout (branch) or whatever)
    ./templates --clean
    ./templates-updates --update

The templates steps are only necessary if there are any changes to the templates files.

For updating the scheduler code:

    cd /root/fedora_openqa
    git pull (or whatever changes)
    python setup.py install
    systemctl restart fedmsg-hub

Updating other components of the scheduling process follow the same pattern: update the code or
package, then remember to restart fedmsg-hub, or the fedmsg consumers won't use the new code.
It's relatively common for the openQA instances to need fedfind updates in advance of them being
pushed to stable, for example when a new compose type is invented and fedfind doesn't understand
it, openQA can end up trying to schedule tests for it, or the scheduler consumer can crash; when
this happens we have to fix and update fedfind on the openQA instances ASAP.


Logging
=======

Just about all useful logging information for all aspects of openQA and the scheduling and
report tools is logged to the journal, except that the Apache server logs may be of interest
in debugging issues related to accessing the web UI or websockets server. To get more detailed
logging from openQA components, change the logging level in `/etc/openqa/openqa.ini` from
'info' to 'debug' and restart the relevant services. Any run of the Ansible plays will reset
this back to 'info'.

Occasionally the test execution logs may be useful in figuring out why all tests are failing
very early, or some specific tests are failing due to an asset going missing, etc. Each job's
execution logs can be accessed through the web UI, on the 'Logs & Assets' tab of the job page;
the files are 'autoinst-log.txt' and 'worker-log.txt'.


Dynamic asset generation (createhdds)
=====================================

Some of the hard disk image file 'assets' used by the openQA tests are created by a tool called
`createhdds`, which is checked out of a git repo to `/root/createhdds` on the servers and also
on some guests. This tool uses `virt-install` and the Python bindings for `libguestfs` to create
various hard disk images the tests need to run. It is usually run in two different ways. The
ansible plays run it in a mode where it will only create expected images that are entirely
missing: this is mainly meant to facilitate initial deployment. The plays also install a file
to `/etc/cron.daily` causing it to be run daily in a mode where it will also recreate images that
are 'too old' (the age-out conditions for images are part of the tool itself).

This process isn't 100% reliable; `virt-install` can sometimes fail, either just quasi-randomly
or every time, in which case the cause of the failure needs to be figured out and fixed so the
affected image can be (re-)built.

The i686 and x86_64 images for each instance are built on the server, as its native arch is
x86_64. The images for other arches are built on one worker host for each arch (nominated by
inclusion in an ansible inventory group that exists for this purpose); those hosts have write
access to the NFS share for this purpose.


Compose check reports (check-compose)
=====================================

An additional ansible role runs on each openQA server, called `check-compose`. This role installs
a tool (also called `check-compose`) and an associated fedmsg consumer. The consumer kicks in
when all openQA tests for any compose finish, and uses the `check-compose` tool to send out an
email report summarizing the results of the tests (well, the production server sends out emails,
the staging server just logs the contents of the report). This role isn't really a part of openQA
proper, but is run on the openQA servers as it seems like as good a place as any to do it. As
with all other fedmsg consumers, if making manual changes or updates to the components, remember
to restart `fedmsg-hub` service afterwards.


Autocloud ResultsDB forwarder (autocloudreporter)
=================================================

An ansible role called `autocloudreporter` also runs on the openQA production server. This has
nothing to do with openQA at all, but is run there for convenience. This role deploys a fedmsg
consumer that listens for fedmsgs indicating that Autocloud (a separate automated test system
which tests cloud images) has completed a test run, then forwards those results to ResultsDB.
