.. title: Taskotron SOP
.. slug: infra-taskotron
.. date: 2014-12-16
.. taxonomy: Contributors/Infrastructure

=============
Taskotron SOP
=============

Run automated tasks to check items in Fedora


Contact Information
===================

Owner
  Fedora QA Devel, Fedora Infrastructure Team

Contact
  #fedora-qa, #fedora-admin, #fedora-noc

Location
  PHX2

Servers
  - taskotron-dev01.qa
  - taskotron-stg01.qa
  - taskotron01.qa
  - taskotron-client*.qa

Purpose
  run automated tasks on fedora components and report results
  of those task runs


Architecture
============

Taskotron is a system for running automated tasks in fedora based on incoming
signals (fedmsgs in our case).

The system is made up of several components:

* trigger
* task execution system (this is a master/slave system, currently using
  buildbot)
* results storage (covered in the resultsdb SOP)
* mirror task git repos


Deploying the Taskotron Master
==============================

The Taskotron master node is responsible for:

1. listening for fedmsgs and scheduling tasks as appropriate
2. coordinating execution of tasks on client nodes

Before doing the initial deployment, a ssh keypair is needed for the taskotron
clients to check tasks out from the git mirror. Generate a password-less
keypair (needed for deploying clients) and put the contents of the public key
in the ``buildslave_ssh_pubkey`` variable.

When running the ``taskotron-[dev,stg,prod]`` group playbook for the first time,
it will fail partway through during buildmaster configuration because
buildmaster initialization is not part of the playbook (it fails if re-run and
no upgrade is needed). Once you hit that failure, run the following on the
taskmaster node as the buildmaster user, run::

  buildbot upgrade-master /srv/buildmaster/master

After running the ``upgrade-master`` command, continue the playbook and it
should run to completion.


Deploying the Taskotron Clients
===============================

Before deploying the taskotron clients, get the host key of the taskotron
master and populate ``buildmaster_pubkey`` in the client's group_vars file.
This will make sure that git checkouts from the master node work without human
intervention.

Deploying the Taskotron clients is a matter of running the proper group
playbook once the variable files are properly filled in. No additional
configuration is needed.


Updating
========

..note::
  It would be wise to update ResultsDB while the Taskotron system is not
  processing jobs - that is covered in the ResultsDB SOP.

There are multiple parts to updating Taskotron: clients, master and git mirrors.

#. Write down the timestamp when you're switching task processing off. After
   the update, you'll search for all fedmsgs since this timestamp. You can see
   the current timestamp like this::

     date -u +%s

#. Stop ``fedmsg-hub`` on the Taskotron master so that no new jobs come in::

     systemctl stop fedmsg-hub

#. Wait for Buildbot to become idle (see its web UI).
#. Stop all buildslaves on all affected clients::

     systemctl stop buildslave@qa*

#. Run the ``update_grokmirror_repos.yml`` playbook on the system to update.
#. Update and reboot the master node.

   * If you're completely reinstalling the master node machine, you'll need
     to back up the Buildbot data first. See `Backup`_ section.

#. Stop ``fedmsg-hub`` on master node once again, because it's set to start at
   boot::

     systemctl stop fedmsg-hub

7. Update and reboot the client nodes.
8. Start the buildslave processes on all client nodes (they aren't set to start
   at boot)::

     cd /home; for BS in qa*; do systemctl start buildslave@$BS; done

#. Process all the fedmsgs that you missed during this update. Run::

     jobrunner --start $TIMESTAMP

   where $TIMESTAMP is the timestamp saved at the beginning of starting the
   update process.

#. Start ``fedmsg-hub`` on Taskotron master::

     systemctl start fedmsg-hub


Making Taskotron idle
=====================

You can use the instructions in the `Updating`_ section also to make Taskotron
idle - skip the update and reboot steps, but turn off ``fedmsg-hub`` and shut
down the ``buildslave@qa*`` services. The buildslave and ``fedmsg-hub``
processes will need to be restarted to un-idle the system but buildbot will
restart anything that was running once the buildslaves come back up.


Backup
======

There are two major things which need to be backed up for Taskotron: job data
and the buildmaster database. Make sure to stop ``buildmaster.service`` before
backing up any of those.

The buildmaster database is a normal postgres dump from the database server.

The job data is stored on the taskotron master node in
``/srv/buildmaster/master/`` directory. The files in ``master/`` are not
important but all subdirectories outside of ``templates/`` and ``public_html/``
are.


Restore from Backup
===================

To restore from backup, load the database dump and restore backed up files to
the provisioned master before starting the ``buildmaster.service``.


Current workarounds
===================

A list of things that are known to be currently broken and how to work around
them.

Buildmaster doesn't start on first attempt
------------------------------------------

When you reboot the server, and have ``buildmaster.service`` configured to
start automatically, it often fails. Running::

  systemctl start buildmaster.service

again fixes the problem and buildmaster starts (you might try several times).

Note: Increasing ``TimeoutStartSec=`` in the unit file doesn't fix this.

nfs/client role fails to execute - nfs-lock service fails to start
------------------------------------------------------------------

Due to `RH bug 1403527 <https://bugzilla.redhat.com/show_bug.cgi?id=1403527>`_
the ``nfs-lock.service`` fails to start, which breaks the ``nfs/client``
ansible role. The workaround is to fix SELinux labels on ``rpcbind``::

  $ restorecon -v /usr/bin/rpcbind
  restorecon reset /usr/bin/rpcbind context system_u:object_r:bin_t:s0->system_u:object_r:rpcbind_exec_t:s0
