.. title: Fedorahosted Cleanup SOP
.. slug: infra-fedorahosted-cleanup
.. date: 2011-10-10
.. taxonomy: Contributors/Infrastructure

======================================
FH-Projects-Cleanup Infrastructure SOP
======================================

Contents

1. Introduction
2. Our first move
3. Removing Project's git repo
4. Removing Trac's project
5. Removing Project's ML
6. FAS Group Removal

Introduction
============

This wiki page will help any sysadmin having a [50]Fedora Hosted Project
completely removed either because the owner requested to have it removed
or for whatever any other issue that would take us to remove a project.
This page covers git, Trac, Mailing List and FAS group clean-up.

Our first move
==============

If you are going to remove a Fedora Hosted's project, please remember to
create a folder into /srv/tmp that should follow the following syntax::

  cd /srv/tmp && mkdir $project-hold-until-xx-xx-xx

where xx-xx-xx should be substituted with the date everything should be
purged away from there. (it happens 14 days after the delete request)

Removing Project's git repo
===========================

Having a git repository removed can be achieved with the following steps::

  ssh uid@fedorahosted.org
  cd /git
  mv $project.git/ /srv/tmp/$project-hold-until-xx-xx-xx/

We're done with git!

Removing Trac's project
=======================

Steps are::

  ssh uid@fedorahosted.org
  cd /srv/web/trac/projects
  mv $project/ /srv/tmp/$project-hold-until-xx-xx-xx/

and...that's all!

Removing Project's ML
=====================

We have two options here:

Delete a list, but keep the archives::

  sudo /usr/lib/mailman/bin/rmlist <listname>

Delete a list and its archives::

  sudo /usr/lib/mailman/bin/rmlist -a <listname>

If you are going to completely remove the Mailing List and its archives,
please make sure the list is empty and there are no subscribers in it.

FAS Group Removal
=================

Not every Fedora sysadmin can have this done. See
[51]ISOP:ACCOUNT_DELETION for information. You may want to remove the
group or simply disable it.
