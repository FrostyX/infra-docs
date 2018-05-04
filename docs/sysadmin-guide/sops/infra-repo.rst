.. title: Infrastructure RPM Repository SOP
.. slug: infra-repo
.. date: 2016-10-12
.. taxonomy: Contributors/Infrastructure

.. _infra-repo:

===========================
Infrastructure Yum Repo SOP
===========================

In some cases RPM's in Fedora need to be rebuilt for the Infrastructure
team to suit our needs. This repo is provided to the public (except for
the RHEL RPMs). Rebuilds go into this repo which are stored on the netapp
and shared via the proxy servers after being built on koji.

For basic instructions, read the standard documentation on Fedora wiki:
- https://fedoraproject.org/wiki/Using_the_Koji_build_system

This document will only outline the differences between the "normal" repos
and the infra repos.


Contents
========

1. Contact Information
2. Building an RPM
3. Tagging an existing build
4. Promoting a staging build
5. Koji package list

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
         PHX [53] https://kojipkgs.fedoraproject.org/repos-dist/
Servers
         koji
	 batcave01 / Proxy Servers
Purpose
	 Provides infrastructure repo for custom Fedora Infrastructure rebuilds

Building an RPM
===============

Building an RPM for Infrastructure is significantly easier then building
an RPM for Fedora. Basically get your SRPM ready, then submit it to koji
for building to the $repo-infra target. (e.g. epel7-infra).

Example::

  rpmbuild --define "dist .el7" -bs test.spec
  koji build epel7-infra test-1.0-1.el7.src.rpm

.. note::
  Remember to build it for every dist / arch you need to deploy it on.

After it has been built, you will see it's tagged as $repo-infra-candidate,
this means that it is a candidate for being signed. The automatic signing
system will pick it up and sign the package for you without any further
intervention. You can track when this is done by checking the build info:
when it is moved from $repo-infra-candidate to $repo-infra-stg, it has been
signed. You can check this on the web interface (look under "Tags"), or via::

  koji buildinfo test-1.0-1.el7.

After the build has been tagged into the $repo-infra-stg tag, tag2distrepo will
automatically create a distrepo task, which will update the repository so that
the package is available on staging hosts.
After this time, you can yum clean all and then install the packages via yum
install or yum update.


Tagging existing builds
=======================

If you already have a real build and want to use it in the infrastructure before
it has landed in stable, you can tag it into the respective infra-candidate tag.
For example, if you have an epel7 build of test2-1.0-1.el7, run::

  koji tag epel7-infra-candidate test2-1.0-1.el7

And then the same autosigning and repogen from the previous section applies.


Promoting a staging build
=========================

After getting autosigned, builds will land in the respective infra-stg tag, for
example epel7-infra-stg.
These tags go into repos that are enabled on staging machines, but not on
production.
If you decide, after testing, that the build is good enough for production, you
can promote it by running::

  koji move epel7-infra-stg epel7-infra test2-1.0-1.el7


Koji package list
=================

If you try to build a package into the infra tags, and koji says something like:
BuildError: package test not in list for tag epel7-infra-candidate
That means that the package has not been added to the list for building in that
particular tag. Either add the package to the respective Fedora/EPEL branches
(this is the preferred method, since we should always aim to get everything
packaged for Fedora/EPEL), or ask a koji admin to add the package to the listing
for the respective tag.

To list koji admins::

  koji list-history -s user_perms --permission=admin --active

For koji admins, they can run::

  koji add-pkg $tag $package --owner=$user
