.. title: Fedora Infrastructure Git Repo SOP
.. slug: infra-git
.. date: 2013-06-17
.. taxonomy: Contributors/Infrastructure

========================
Infrastructure Git Repos
========================

Setting up an infrastructure git repo - and the push mechanisms for the
magicks

We have a number of git repos (in /git on lockbox) that manage files
for puppet, our docs, our common host info database and our kickstarts
This is a doc on how to setup a new one of these, if it is needed.

Contact Information
===================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-admin, sysadmin-main
Location
	Phoenix
Servers
  lockbox01.phx2.fedoraproject.org,
  lockbox-comm01.qa.fedoraproject.org


Steps
======
Create the bare repo::

  make $git_dir
  setfacl -m d:g:$yourgroup:rwx -m d:g:$othergroup:rwx  \
   -m g:$yourgroup:rwx -m g:$othergroup:rwx $git_dir

  cd $git_dir
  git init --bare


edit up config - add these lines to the bottom::

  [hooks]
  # (normallysysadmin-members@fedoraproject.org)
  mailinglist = emailaddress@yourdomain.org 
  emailprefix = 
  maildomain = fedoraproject.org
  reposource = /path/to/this/dir
  repodest = /path/to/where/you/want/the/files/dumped


edit up description - make it something useful::


  cd hooks
  rm -f *.sample
  cp hooks from /git/infra-docs/hooks/ on lockbox01 to this path

modify sudoers to allow users in whatever groups can commit to 
this repo can run /usr/local/bin/syncgittree.sh w/o inputting a password


If you want zodbot commit notifications
=======================================

The zodbot announce script is in the puppet 'scripts' module.  It is
*important* that you symlink it in, and not copy it.  We only want to have to
change it once place.

::
  
  cd $gitdir/hooks
  ln -s /usr/local/bin/zodbot-announce-commits.py .
  $EDITOR ./update

Add these lines::

  reposource=$(git config hooks.reposource)
  zodbot_channel=$(git config hooks.zodbotchannel)
  python $reposource/hooks/zodbot-announce-commits.py $reposource $zodbot_channel $oldrev $newrev

Stop adding beyond this point::

  cd ..
  $EDITOR ./config

Add the following to the [hooks] section you made above.::

  zodbotchannel = "#fedora-noc"  # (Or whatever other channel you need)

Stop adding beyond this point

Then test the heck out of it. :)
