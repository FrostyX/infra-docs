.. title: Fedorahosted Repository Setup SOP
.. slug: infra-fedorahosted-repo-setup
.. date: 2014-09-24
.. taxonomy: Contributors/Infrastructure

=======================
Hosted repository setup
=======================

Fedora provides SCM repositories for open source projects.

Contents

1. Mercurial Repository

  3. Repo Setup
  2. Commit Mail

2. Git Repository

   1. Repo Setup
   2. Commit Mail

3. Bazaar Repository
4. SVN Repository

  1. Repo Setup
  2. Commit Mail

Mercurial Repository
====================

You'll need to know three things in order to start the mercurial
repository.

PROJECTNAME
  what the project wants to be called.

OLDURL
  how to access the project's current sourcecode in their
  mercurial repository.

PROJECTGROUP
  the group setup in the account system for readwrite
  access to the repository.

Repo Setup
----------

The Mercurial repository lives on the hosted server. Access it by logging
into hosted1 Then follow these steps:

1. Fetch latest content from the FAS Database.::

   $ fasClient -i -f

2. Create the repo::

    $ cd /hg
    $ sudo hg clone -U $OLDURL $PROJECTNAME (or sudo mkdir $PROJECTNAME; cd $PROJECTNAME; sudo hg init)
    $ sudo find $PROJECTNAME -type d -exec chmod g+s \{\} \;
    $ sudo chmod -R g+w $PROJECTNAME
    $ sudo chown -R root:$PROJECTGROUP $PROJECTNAME

This should setup all the files needed for the repository.

Commit Mail
-----------

The Mercurial Notify extension can be used to send out email when
commits are pushed to a Mecurial repository. To enable notifications,
create the file ``/hg/$PROJECTNAME/.hg/hgrc``::

  [extensions]
  hgext.notify =

  [hooks]
  changegroup.notify = python:hgext.notify.hook

  [email]
  from = admin@fedoraproject.org

  [smtp]
  host = localhost

  [web]
  baseurl = http://hg.fedorahosted.org/hg

  [notify]
  sources = serve push pull bundle
  test = False
  config = /hg/$PROJECTNAME/.hg/subscriptions
  maxdiff = -1

And the file ``/hg/$PROJECTNAME/.hg/subscriptions``::

  [usersubs]

  user@host = *

  [reposubs]

Git Repository
--------------

You'll need to know several things in order to start the git repository.


PROJECTNAME
	what the project wants to be called.

OLDURL
	how to access the project's current source code in their git repository.

PROJECTGROUP
	the group setup in the account system for write access to the repository.

COMMITLIST
	comma-separated list of email addresses for commits (optional)

DESCRIPTION
	description of the project (optional)

PROJECTOWNER
	the FAS username of the project owner

Repo Setup
----------

The git repository lives on the hosted server. Access it by logging into
hosted1 Then follow these steps:

Fetch latest content from the FAS Database.::

  $ sudo fasClient -i -f

  $ cd /git

Clone an existing repository::

    $ sudo git clone --bare $OLDURL $PROJECTNAME.git
    $ cd $PROJECTNAME.git
    $ sudo git config core.sharedRepository true
    $ #
    $ ## or
    $ #
    $ # Create a new repository:
    $ sudo mkdir $PROJECTNAME.git
    $ cd $PROJECTNAME.git
    $ sudo git init --bare --shared=true

Give the repository a nice description for gitweb::

  $ echo $DESCRIPTION | sudo tee description > /dev/null

Setup and run post-update hook.

..note::
  We symlink this because /git is on a filesystem with noexec set)

::

  $ sudo ln -svf /usr/share/git-core/templates/hooks/post-update.sample ./hooks/post-update
  $ sudo git update-server-info

Ensure ownership and modes are correct::

  $ sudo find -type d -exec chmod g+s \{\} \;
  $ sudo find -perm /u+w -a ! -perm /g+w -exec chmod g+w \{\} \;
  $ sudo chown -R $PROJECTOWNER:$PROJECTGROUP .

This should setup all the files needed for the repository. The repository
owner can push changes into the repo by running::

  $ git push ssh://git.fedorahosted.org/git/$PROJECTNAME.git/ master

from within their local git repository.

Commit Mail
-----------

If they want commit mail, then there are a couple of additional steps.::

  $ cd /git/$PROJECTNAME.git
  $ sudo git config hooks.mailinglist $COMMITLIST
  $ sudo git config hooks.maildomain fedoraproject.org
  $ sudo git config hooks.emailprefix "[$PROJECTNAME]"
  $ sudo git config hooks.repouri "http://git.fedorahosted.org/cgit/$PROJECTNAME.git"
  $ sudo ln -svf /usr/share/git-core/post-receive-chained ./hooks/post-receive
  $ sudo mkdir ./hooks/post-receive-chained.d
  $ sudo ln -svf /usr/local/bin/git-notifier ./hooks/post-receive-chained.d/post-receive-email
  $ sudo ln -svf /usr/local/share/git/hooks/post-receive-fedorahosted-fedmsg ./hooks/post-receive-chained.d/post-receive-fedmsg

Bazaar Repository
=================
You'll need to know three things in order to start a bazaar repository.


PROJECTNAME
	what the project wants to be called.

OLDBRANCHURL
  how to access the project's current sourcecode in
  their previous bazaar repository. Note that a project may have
  multiple branches that they want to import. Each branch will have a
  separate URL. (The project can import the new branches after the
  repository is created if they want.)

PROJECTGROUP
  the group setup in the account system for readwrite
  access to the repository.

Repo Setup
----------

The bzr repository lives on the hosted server. Access it by logging into
hosted1 then follow these steps:

The first stage is to create the Bazaar repository.

Fetch latest content from the FAS Database.::

  $ fasClient -i -f

  $ cd /srv/bzr/
  $ # This creates a Bazaar repository which has shared storage between branches
  $ sudo bzr init-repo $PROJECTNAME --no-trees
  $ cd $PROJECTNAME
  $ sudo bzr branch $OLDURL
  $ sudo bzr branch $OLDURL2
  $ # [...]
  $ sudo bzr branch $OLDURLN
  $ cd ..
  $ sudo find $PROJECTNAME -type d -exec chmod g+s \{\} \;
  $ sudo chmod -R g+w $PROJECTNAME
  $ sudo chown -R root:$PROJECTGROUP $PROJECTNAME

This should be all that is needed. To checkout run::

  bzr init-repo $MYLOCALPROJECTREPO
  cd $MYLOCALPROJECTREPO
  bzr branch bzr+ssh://bzr.fedorahosted.org/bzr/$PROJECTNAME/$BRANCHNAME
  bzr branch bzr://bzr.fedorahosted.org/bzr/$PROJECTNAME/$BRANCHNAME/

.. note::
  If the end user checks out a branch without creating their own
  repository they will need to create a local working tree by doing the
  following::

    cd $BRANCHNAME
    bzr checkout --lightweight

SVN Repository
==============

You'll need to know two things in order to start a svn repository.


PROJECTNAME
	what the project wants to be called.

PROJECTGROUP
  The Fedora account system group with read-write
  access.

COMMITLIST
  comma-separated list of email addresses for commits
  (optional)

Repo Setup
----------

SVN lives on the hosted server. Access it by logging into hosted1. Then
run the following steps:

Fetch latest content from the FAS Database.::

  $ fasClient -i -f

Create the repo::

  $ cd /svn/
  $ sudo svnadmin create $PROJECTNAME
  $ cd $PROJECTNAME
  $ sudo chgrp -R $PROJECTGROUP .
  $ sudo chmod -R g+w .
  $ sudo find -type d -exec chmod g+s \{\} \;

This should be all that is needed. To checkout run::

  svn co svn+ssh://svn.fedorahosted.org/svn/$PROJECTNAME

Commit Mail
-----------

If they want commit mail, then there are a couple of additional steps.::

  $ echo $COMMITLIST | sudo tee ./commit-list > /dev/null
  $ sudo ln -sv /usr/bin/fedora-svn-commit-mail-hook ./hooks/post-commit
