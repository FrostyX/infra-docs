.. title: Fedora Infrastructure Git Repo SOP
.. slug: infra-git
.. date: 2013-06-17
.. taxonomy: Contributors/Infrastructure

========================
Infrastructure Git Repos
========================

Setting up an infrastructure git repo - and the push mechanisms for the
magicks

We have a number of git repos (in /git on batcave) that manage files
for ansible, our docs, our common host info database and our kickstarts
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
  batcave01.phx2.fedoraproject.org,
  batcave-comm01.qa.fedoraproject.org


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
  cp hooks from /git/infra-docs/hooks/ on batcave01 to this path

modify sudoers to allow users in whatever groups can commit to
this repo can run /usr/local/bin/syncgittree.sh w/o inputting a password
