.. title: Fedorahosted Repository Migration SOP
.. slug: infra-fedorahosted-migration
.. date: 2011-12-14
.. taxonomy: Contributors/Infrastructure

=======================
Fedorahosted migrations
=======================

Migrating hosted repositories to that of another type.

Contents
========
1. Contact Information
2. Description
3. SVN to GIT migration

  1. Questions left to be answered with this SOP

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-hosted

Location
	Serverbeach

Servers
	hosted1, hosted2

Purpose
	Migrate hosted SCM repositories to that of another SCM.

Description
===========

fedorahosted.org can be used to host open source projects. Occasionally
those projects want to change the SCM they utilize. This document provides
documentation for doing so.

1. An scm for maintaining the code. The currently supported scm's include
   Mercurial, Git, Bazaar, or SVN. Note: There is no cvs
2. A trac instance, which provides a mini-wiki for hosting information
   and also provides a ticketing system.
3. A mailing list

.. important::
  This page is for administrators only. People wishing to request a hosted
  project should use the [50]Ticketing System ; see the
  new project request template. (Requires Fedora Account)

SVN to GIT migration
====================

FAS User Prep
--------------

Currently you must manually generate $PROJECTNAME-users.txt by grabbing a
list of people in the FAS group - and recording them in th following
format::

  $fasusername = FirstName LastName <$emailaddress>

This is error prone, and will stop the git-svn fetch below if an author
appears that doesn't exist in the list of users.::

  svn log --quiet | awk '/^r/ {print $3}' | sort -u

The above will generate a list of users in the svn repo.

If all users are FAS users you can use the following script to create a
users file (written by tmz (Todd Zullinger)::

  #!/bin/bash

  if [ -z "$1" ]; then
   echo "usage: $0 <svn repo>" >&2
   exit 1
  fi

  svnurl=file:///svn/$1

  if ! svn info $svnurl &>/dev/null; then
   echo "$1 is not a valid svn repo." >&2
  fi

  svn log -q $svnurl | awk '/^r[0-9]+/ {print $3}' | sort -u | while read user; do
   name=$( (getent passwd $user 2>/dev/null | awk -F: '{print $5}') || '' )
   [ -z "$name" ] && name=$user
   email="$user@fedoraproject.org"
   echo "$user=$name <$email>"
  done

Doing the conversion
---------------------

1. Log into hosted1
2. Make a temporary directory to convert the repos in::

      $ sudo mkdir /tmp/tmp-$PROJECTNAME.git

      $ cd /tmp/tmp-$PROJECTNAME.git

3. Create an git repo ready to receive migrated SVN data::

      $ sudo git-svn init http://svn.fedorahosted.org/svn/$PROJECTNAME --no-metadata

4. Tell git to fetch and convert the repository::

      $ git svn fetch

    .. note::
      This creation of a temporary repository is necessary because SVN leaves a
      number of items floating around that git can ignore, and we want those
      essentially ignored.

5. From here, you'll wanted to follow [53]Creating a new git repo as if
    cloning an existing git repository to Fedorahosted.

6. After that process is done - kindly remove the temporary repo that was created::

      $ sudo rm -rf /tmp/tmp-$PROJECTNAME.git

Doing the converstion (alternate)
---------------------------------

Alternately, here's another way to do this (tmz):

Setup a working dir::

  [tmz@hosted1 tmp (master)]$ mkdir im-chooser-conversion && cd im-chooser-conversion

Create authors file mapping svn usernames to Name <email> form git uses.::

  [tmz@hosted1 im-chooser-conversion (master)]$ ~tmz/svn-to-git-authors im-chooser > authors

Convert svn to git::

  [tmz@hosted1 im-chooser-conversion (master)]$ git svn clone -s -A authors --no-metadata file:///svn/im-chooser

Move svn branches and tags into proper locations for the new git repo.
(git-svn leaves them as 'remote' branches/tags.)::

  [tmz@hosted1 im-chooser-conversion (master)]$ cd im-chooser
  [tmz@hosted1 im-chooser (master)]$ mv .git/refs/remotes/tags/* .git/refs/tags/ && rmdir .git/refs/remotes/tags
  [tmz@hosted1 im-chooser (master)]$ mv .git/refs/remotes/* .git/refs/heads/

Now 'git branch' and 'git tag' should display the branches/tags.

Create a bare repo from the converted git repo.
Using ``file://$(pwd)`` here ensures that git copies all objects to the new bare repo.::

  [tmz@hosted1 im-chooser-conversion (master)]$ git clone --bare --shared file://$(pwd)/im-chooser im-chooser.git

Follow the steps in https://fedoraproject.org/wiki/Hosted_repository_setup to
finish setting proper modes and permissions for the repo.  Don't forget to
update the description file.

.. note::
  This still leaves moving the converted bare repo (im-chooser.git) to /git
  and fixing up the user/group.

Questions left to be answered with this SOP
============================================

* Obviously we need to have requestor review the migration and confirm
  it's ok.
* Do we then delete the old SCM contents?
* Do we need to change the FAS-group type to grant them access to
  pull/push from it?
