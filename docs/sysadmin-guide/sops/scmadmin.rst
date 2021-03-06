.. title: Infrastructure SCM Admin SOP
.. slug: infra-scm-admin
.. date: 2015-01-01
.. taxonomy: Contributors/Infrastructure

=============
SCM Admin SOP
=============

.. warning:: Most information here (probably 1.4 and later) is not updated for
  pkgdb2 and therefore not correct anymore.

Contents
========

1. Creating New Packages

  1. Obtaining process-git-requests
  2. Prerequisites
  3. Running the script
  4. Steps for manual processing

    1. Using pkgdb-client
    2. Using pkgdb2branch
    3. Update Koji

  5. Helper Scripts

    1. mkbranchwrapper
    2. setup_package

  6. Pseudo Users for SIGs

2. Deprecate Packages
3. Undeprecate Packages
4. Performing mass comaintainer requests

Creating New Packages
=====================
 
Package creation is mostly automatic and most details are handled by a script.

Obtaining process-git-requests
------------------------------

The script is not currently packaged; lives in the rel-eng
git repository. You can check it out with::

  git clone https://git.fedorahosted.org/git/releng

and keep this up to date by running::

  git pull

occasionally somewhere in the checked-out tree occasionally before
processing new requests.

The script lives in ``scripts/process-git-requests``.

Prerequisites
-------------
   
You must have the python-bugzilla and python-fedora packages installed.

Before running process-git-requests, you should run::

  bugzilla login

The "Username" you will be prompted for is the email address attached to
your bugzilla account. This will obtain a cookie so that the script can
update bugzilla tickets. The cookie is good for quite some time (at least
a month); if you wish to remove it, delete the ``~/.bugzillacookies`` file.

It is also advantageous to have your Fedora ssh key loaded so that you can
ssh into pkgs.fedoraproject.org without being prompted for a password.

It perhaps goes without saying that you will need unfirewalled and
unproxied access to ports 22, 80 and 443 on various Fedora machines.

Running the script
------------------

Simply execute the process-git-requests script and follow the prompts. It
can provide the text of all comments in the bugzilla ticket for inspection
and will perform various useful checks on the ticket and the included SCM
request. If there are warnings present, you will need to accept them
before being allowed to process the request.

Note that the script only looks at the final request in a ticket; this
permits users to tack on a new request at any time and re-raise the
fedora-cvs flag. Packagers do not always understand this, though, so it is
necessary to read through the ticket contents to make sure that's the
request matches reality.

After a request has been accepted, the script will create the package in
pkgdb (which may require your password) and attempt to log into the SCM
server to create the repository. If this does not succeed, the package
name is saved and when you finish processing a command line will be output
with instructions on creating the repositories manually. If you hit Crtl-C
or the script otherwise aborts, you may miss this information. If so, see
below for information on running pkgdb2branch.py on the SCM server; you
will need to run it for each package you created.

Steps for manual processing
---------------------------

It is still useful to document the process of handling these requests
manually in the case that process-git-requests has issues.

1. Check Bugzilla Ticket to make sure it looks ok
2. Add the package information to the packagedb with pkgdb-client
3. Use pkgdb2branch to create the branches on the cvs server
    
  .. warning:: Do not run multiple instances of pkgdb2branch in parallel!
    This will cause them to fail due to mismatching 'modules' files. It's not
    a good idea to run addpackage, mkbranchwrapper, or setup_package by
    themselves as it could lead to packages that don't match their packagedb
    entry.

4. Update koji.

Using pkgdb-client
``````````````````

Use pkgdb-client to update the pkgdb with new information. For instance,
to add a new package:::

  pkgdb-client edit -u toshio -o terjeros \
   -d 'Python module to extract EXIF information' \
   -b F-10 -b F-11 -b devel python-exif

To update that package later and add someone to the initialcclist do::

  pkgdb-client edit -u toshio -c kevin python-exif

To add a new branch for a package::

  pkgdb-client edit -u toshio -b F-10 -b EL-5 python-exif

To allow provenpackager to edit a branch::

  pkgdb-client edit -u toshio -b devel -a provenpackager python-exif

To remove provenpackager commit rights on a branch::

  pkgdb-client edit -u toshio -b EL-5 -b EL-4 -r provenpackager python-exif

More options can be found by running ``pkgdb-client --help``

You must be in the cvsadmin group to use pkgdb-client. It can be run on a
non-Fedora Infrastructure box if you set the PACKAGEDBURL environment
variable to the public URL::

  export PACKAGEDBURL=https://admin.fedoraproject.org/pkgdb

.. note::
   You may be asked to CC fedora-perl-devel-list on a perl package. This can
   be done with the username "perl-sig". This is presently a user, not a
   group so it cannot be used as an owner or comaintainer, only for CC.

Using pkgdb2branch
------------------

Use pkgdb2branch.py to create branches for a package. pkgdb2branch.py
takes a list of package names on the command line and creates the branches
that are specified in the packagedb. The script lives in /usr/local/bin on
the SCM server (pkgs.fedoraproject.org) and must be run there. 

For instance, ``pkgdb2branch.py python-exif qa-assistant`` will create branches 
specified in the packagedb for python-exif and qa-assistant.

pkgdb2branch can only be run from pkgs.fedoraproject.org.

Update Koji
-----------

Optionally you can synchronize pkgdb and koji by hand: it is done
automatically hourly by a cronjob. There is a script for this in the
admin/ directory of the CVSROOT module.

Since dist-f13 and later inherit from dist-f12, and currently dist-f12 is
the basis of our stack, it's easiest to just call::

  ./owner-sync-pkgdb dist-f12

Just run ``./owners-sync-pkgdb`` for usage output.

This script requires that you have a properly configured koji client
installed.

owner-sync-pkgdb requires the koji client libraries which are not
available on the cvs server. So you need to run this from one of your
machines.

Helper Scripts
==============
 
These scripts are invoked by the scripts above, doing some of the heavy
lifting. They should not ordinarily be called on their own.

mkbranchwrapper
---------------

``/usr/local/bin/mkbranchwrapper`` is a shell script which takes a list of
packages and branches. For instance::

  mkbranchwrapper foo bar EL-5 F-11

will create modules foo and bar for devel if they don't exist and branch
them for the other 4 branches passed to the script. If the devel branch
exists then it just branches. If there is no branches passed the module is
created in devel only.

``mkbranchwrapper`` has to be run from cvs-int.

.. important:: mkbranchwrapper is not used by any current programs. Use pkgdb2branch  instead.

setup_package
-------------

``setup_package`` creates a new blank module in devel only. It can be run from
any host. To create a new package run::

  setup_package foo

setup_package needs to be called once for each package. it could be
wrapped in a shell script similar to::

  #!/bin/bash

  PACKAGES=""

  for arg in $@; do
    PACKAGES="$PACKAGES $arg"
  done

  echo "packages=$PACKAGES"

  for package in $PACKAGES; do
    ~/bin/setup_package $package
  done


then call the script with all branches after it.

.. note:: setup_package is currently called from pkgdb2branch.

Pseudo Users for SIGs
---------------------

See [62]Package_SCM_admin_requests#Pseudo-users_for_SIGs for the current list.

Deprecate Packages
------------------

Any packager can deprecate a package. click on the deprecate package
button for the package in the webui. There's currently no ``pkgdb-client``
command to deprecate a package.

Undeprecate Packages
--------------------

Any cvsadmin can undeprecate a package. Simply use pkgdb-client to assign
an owner and the package will be undeprecated::

  pkgdb-client -o toshio -b devel qa-assistant

As a cvsadmin you can also log into the pkgdb webui and click on the
unretire package button. Once clicked, the package will be orphaned rather
than deprecated.

Performing mass comaintainer requests
-------------------------------------

* Confirm that the requestor has 'approveacls' on all packages they wish
  to operate on. If they do not, they MUST request the change via FESCo.

* Mail maintainers/co-maintainers affected by the change to inform them
  of who requested the change and why.

* Download a copy of this script:
  http://git.fedorahosted.org/git/?p=fedora-infrastructure.git;a=blob;f=scripts/pkgdb_bulk_comaint/comaint.py;hb=HEAD

* Edit the script to have the proper package owners and package name
  pattern.

* Edit the script to have the proper new comaintainers.

* Ask someone in ``sysadmin-web`` to disable email sending on bapp01 for the
  pkgdb (following the instructions in comments in the script)

* Copy the script to an infrastructure host (like cvs01) that can
  contact bapp01 and run it.

