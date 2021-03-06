.. title: ReviewBoard Infrastucture SOP
.. slug: infra-reviewboard
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

==============================
ReviewBoard Infrastructure SOP
==============================

Review Board is a powerful web-based code review tool that offers
developers an easy way to handle code reviews. It scales well from small
projects to large companies and offers a variety of tools to take much of
the stress and time out of the code review process.

Contents
========

1.  Contact Information
2. File Locations
3. Troubleshooting and Resolution
  
  * Restarting

4. Create a new repository in ReviewBoard

  * Creating a new git repository
  * Creating a new bzr repository
  * Create a default reviewer for a repository

Contact Information
===================

Owner: 
  Fedora Infrastructure Team

Contact: 
  #fedora-admin, sysadmin-main, sysadmin-hosted

Location: 
  ServerBeach

Servers: 
  hosted[1-2]

Purpose: 
  Provide our fedorahosted users a way to review code.

File Locations
==============
Main Config File:
   hosted[1-2]:/srv/reviewboard/conf/settings_local.py 

ReviewBoard:
   hosted[1-2]:/etc/httpd/conf.d/fedorahosted.org/reviewboard.conf 
 
Upstream:   
  https://fedorahosted.org/reviewboard/

Troubleshooting and Resolution
==============================

Restarting
----------

After an update, to restart reviewboard just restart apache. Doing a
service httpd stop and then a service httpd start should do it.

Create a new repository in ReviewBoard
======================================

Creating a new git repository
-----------------------------

1. Enter the admin interface. If you have admin privilege, a link will be
    visible in the upper-right corner of the dashboard.
2. In the admin dashboard click "Add" next to "Repositories"
3. For the name, enter the Fedora Hosted project short name. (e.g. if the
    project is [53]https://fedorahosted.org/sssd, then the repository name
    should be sssd)
4. "Show this repository" must be checked.
5. Hosting service is "Custom"
6. Repository type is Git
7. Path should be /srv/git/project_short_name.git 
   (e.g. /srv/git/sssd.git)
8. Mirror path should be
    git://git.fedorahosted.org/git/project_short_name.git

.. note:: Mirror path is used by client tools such as post-review to
    determine to which repository a submission belongs

9. Raw file URL mask should be left blank
10. Username and Password should both be left blank
11. The bug tracker URL may vary from project to project, but if they are
    using the Fedora Hosted Trac bugtracker, it should be

    * Type: Trac
    * Bug Tracker URL: [54]https://fedorahosted.org/project_short_name
        (e.g. [55]https://fedorahosted.org/sssd)

12. Do not set a Bug Tracker URL

Creating a new bzr repository
-----------------------------
1. Go to the admin dashboard to [56]add a new repository.
2. For the name, enter the Fedora Hosted project short name. (e.g. if the
    project is [57]https://fedorahosted.org/kitchen, then the repository
    name should be kitchen)
3. "Show this repository" must be checked.
4. Hosting service is "Custom"
5. Repository type is Bazaar
6. Path should be /srv/git/project_short_name/branch_name 
   (e.g. /srv/bzr/kitchen/devel) -- reviewboard doesn't understand how to work
   with repository conventions; it just works on branches.
7. Mirror path should be
    bzr://bzr.fedorahosted.org/bzr/project_short_name/branch_name

.. note:: Mirror path is used by client tools such as post-review to
            determine to which repository a submission belongs

8. Username and Password should both be left blank
9. The bug tracker URL may vary from project to project, but if they are
    using the Fedora Hosted Trac bugtracker, it should be

    * Type: Trac
    * Bug Tracker URL: [58]https://fedorahosted.org/project_short_name
      (e.g. [59]https://fedorahosted.org/kitchen)

10. Do not set a Bug Tracker URL

Create a default reviewer for a repository
------------------------------------------

Reviews should be sent to the project development mailing list unless
otherwise requested.

1. Enter the admin interface. If you have admin privilege, a link will be
   visible in the upper-right corner of the dashboard.
2. In the admin dashboard click "Add" next to "Review Groups"
3. Enter the following values:

  * Name: The project short name
  * Display Name: project_short_name Review Group
  * Mailing List: Development discussion list for the project

4. Do not select any users
5. Return to the main admin dashboard and click on "Add" next to "Default
    Reviewers"
6. Enter the following values:

  * Name: Something unique and sensible
  * File Regular Expression: enter '.*' (without the quotes)

.. note:: This means that by default, the mailing list should receive
          email for reviews of all files in the repository

7. Under "Default groups", select the group you created above and click
    the arrow pointing right.
8. Do not select any default people
9. Under "Repositories", select the repository added above and click the
    arrow pointing right.
10. Save your changes.
