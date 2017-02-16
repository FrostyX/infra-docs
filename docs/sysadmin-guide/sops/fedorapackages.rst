.. title: Fedora Packages SOP
.. slug: infra-fedora-packages
.. date: 2012-02-23
.. taxonomy: Contributors/Infrastructure

===================
Fedora Packages SOP
===================

This SOP is for the Fedora Packages web application.
https://community.dev.fedoraproject.org/packages

Contents
========

1. Contact Information
2. Building a new release
3. Deploying to the development server
4. Hotfixing
5. Checking for AGPL violations

Contact Information
===================

Owner
	Luke Macken

Contact
	lmacken@redhat.com

Location
	PHX2

Servers
	community01.dev

Purpose
	Web interface for package information

Building a new release
======================
There is a helper script that lives in the fedoracommunity git repository
that automatically handles spinning up a new release, building it in mock, and
scping it to batcave. First, edit the version/release in the specfile and
setup.py, then run:::

  ./release <version> <release>

Deploying to the development server:
=====================================

There is a script in the fedoracommunity git repository called
'fcomm-dev-update' that you must first copy to the ansible server. Then you run
it with the same arguments as the release script. This tool will sign the
RPMs, copy them into the infrastructure testing repo, update the repodata,
and then run a bunch of func commands to update the package on the dev server.

::

  ./fcomm-dev-release <version> <release>

Hotfixing
=========
If you wish to make a hotfix to the Fedora Packages application, simply
make your change in your local git repository, and then perform the building &
deployment steps above. This will still work even if you do not wish to commit
& push your change back upstream.

In order to ensure AGPL compliance, we DO NOT do ansible based hotfixing for
Fedora Packages.

Checking for AGPL violations
============================

To remain AGPL compliant, we must ensure that all modifications to the code
are made available in the SRPM that we link to in the footer of the
application. You can easily query our app servers to determine if any AGPL
violating code modifications have been made to the package.::

  func-command --host="*app*" --host="community*" "rpm -V fedoracommunity"

You can safely ignore any changes to non-code files in the output. If any
violations are found, the Infrastructure Team should be notified immediately.