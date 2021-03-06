.. title: Fedorahosted Infrastructure SOP
.. slug: infra-fedorahosted
.. date: 2014-09-22
.. taxonomy: Contributors/Infrastructure

===============================
Fedorahosted Infrastructure SOP
===============================

Provide hosting place for open source projects.

.. important::
  This page is for administrators only. People wishing to request a hosted
  project should use the Ticketing System ; see the
  new project request template. (Requires Fedora Account)

Contact Information
===================

Owner
  Fedora Infrastructure Team
Contact
  #fedora-admin, sysadmin-hosted
Location
  Serverbeach
Servers
  hosted03, hosted04
Purpose
  Provide hosting place for open source projects

Description
===========

fedorahosted.org can be used to host open source projects. It provides the
following facilities:

1. An scm for maintaining the code. The currently supported SCMs include
    Mercurial, Git, Bazaar, or SVN. There is no cvs.
2. A trac instance, which provides a mini-wiki for hosting information
    and also provides a ticketing system.
3. A mailing list

How to setup a new hosted project
=================================

1. Create source group in Fedora Account System of the form <scm><project>
    ex ``gitepel``, ``svnkernel``, etc

2. Create source repo <see hosted repository setup SOP>

3. Log into hosted03

4. Create new project space::

    sudo /usr/local/bin/hosted-setup.sh <project name> <project admin> <scm type>

  * <project name> must use the same case as the scm repo
  * You're likely to end up with::

      'Command failed: columns username, action are not unique'

    this can be safely ignored as this only tries to tell you
    that you are giving admin access to a person already
    having admin access.

5. If a mailing list is desired, follow the directions for the mailman SOP.

How to import data from a cvs repo into git repo
================================================

Often users request their git repos to be imported from an existing cvs
repo. This is a two step process as follows::

  git-cvsimport -v -d :pserver:anonymous@cvs.fedoraproject.org/cvs/docs -C <dir> <cvs_module>

  sudo git clone --bare --no-hardlinks </pathto/cvsimported/repo> /git/<git_dir>.git/

Example::

  git-cvsimport -v -d :pserver:anonymous@cvs.fedoraproject.org/cvs/docs -C translation-quick-start-guide translation-quick-start-guide
  sudo git clone --bare --no-hardlinks translation-quick-start-guide/ /git/translation-quick-start-guide.git/

.. note::

   Note that our git repos disallow non-fast-forward pushes by default. This
   default makes the most sense, but sometimes, users understand the impact
   of doing so, but still wish to make such a push.

   To enable this temporarily, edit the config file inside of the git repo,
   and make sure that receive.denyNonFastforwards is set to false. Make sure
   to reenable this once the user has finished their push.

How to allow a project to redirect parts of their release tree
==============================================================

A project may want to host parts of their release tree elsewhere (for
instance, moving docs from hosting inside of the fedorhosted release tree
to an external service).  To do that, modify:::

  configs/web/fedorahosted.org/release.conf

Adding a new Directory section like this::

      # Allow python-fedora project to redirect documentation/release tree elsewhere
      <Directory /srv/web/releases/p/y/python-fedora>
        AllowOverride FileInfo
      </Directory>

Then tell the project that they can create a .htaccess file with the
Redirect (Note that the release tree can be reached by two URLs so you need to
redirect both of them)::

     Redirect permanent /releases/p/y/python-fedora/doc http://pythonhosted.org/python-fedora
     Redirect permanent /released/python-fedora/doc/ http://pythonhosted.org/python-fedora
