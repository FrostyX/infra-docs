.. title: Fedorahosted Project Rename SOP
.. slug: infra-fedorahosted-rename
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

===============================
FedoraHosted Project Rename SOP
===============================

This describes the steps necessary to rename a project in Fedora Hosted.

Contents
========

1. Rename the Trac instance
2. Rename the git / svn / hg / ... directory
3. Rename any old releases directories
4. Rename the group in FAS

Rename the Trac instance
=========================

::

  cd /srv/web/trac/projects
  mv oldname newname
  cd newname/conf
  sed -i -e 's/oldname/newname/' trac.ini
  cd ..
  sudo -u apache trac-admin .
  resync

Rename the git / svn / hg / ... directory
=========================================

::

  cd /git
  mv oldname.git newname.git

Rename any old releases directories
===================================

::

  cd /srv/web/releases/o/l/oldname

somehow, the newname releases dir gets created; if there were old releases, move them to the new location.

Rename the group in FAS
=======================

.. note::
   Don't blindly rename
   fedorahosted groups are usually safe to rename. If the old group could be
   present in other apps/configs, though, (like provenpackagers, perl-sig,
   etc) do not rename them. The other apps would need to have the group name
   updated there as well to make this safe.

::

  ssh db2
  sudo -u postgres psql fas2

::

  BEGIN;
  select * from groups where name = '$OLDNAME';
  update groups set name = '$NEWNAME' where name = '$OLDNAME';

* Check that only one row was modified::

    select * from groups where name in ('$OLDNAME', '$NEWNAME');

* Check that there's only one row and the name == $NEWNAME

* If incorrect, do ROLLBACK; instead of commit::

    COMMIT;

.. warning:: Don't delete groups
   If, for some reason, you end up with a group in FAS that was a typo but it
   doesn't conflict with anything else, don't delete it without talking to
   other admins on fedora-infrastructure-list. The numeric group ids could be
   present on a filesystem somewhere and removing the group could eventually
   lead to the id being allocated to some other group which would give
   unintended people access to the files. As a group we can figure out what
   hosts and files need to be checked for this issue if a delete is needed.
