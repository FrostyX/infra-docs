.. title: Badges SOP
.. slug: infra-badges
.. date: 2018-07-29
.. taxonomy: Contributors/Infrastructure

##########
badges SOP
##########

  Fedora Badges - a recognition system for contributors


*******************
Contact Information
*******************

Owner:
  Fedora Infrastructure Team, sysadmin-badges
Contact: 
  #fedora-apps, #fedora-admin, #fedora-noc
Servers:
  badges-web0*, badges-backend0*
Purpose:
  Award "badges" to Fedora Contributors


***********
Description
***********

The badge-awarding back-end daemon, `fedbadges`_, wakes up when it receives a `fedmsg <http://fedmsg.com/en/stable/>`_ event.
It compares that message and the history in `datanommer <https://github.com/fedora-infra/datanommer>`_ against a series of rules.
If a contributor matches the criteria described in one of those rules, then they are awarded a badge.

The front-end is a web application called `Tahrir`_.
It mostly is just an interface for users to look at their badges.
However, it also has an admin interface for manually adding new badges, awarding badges, and creating "invitations" (QR codes) for badges.

The badge-awarding back-end daemon, `fedbadges`_, runs on the ``badges-backend01`` node.
The process is ``fedmsg-hub`` and logs are in ``/var/log/fedmsg/fedmsg-hub.log``.

The front-end runs under ``apache/mod_wsgi`` on ``badges-web0{1,2}`` nodes.

Upstream documentation
======================

- For a detailed description of how the fedbadges daemon works, see the `fedbadges README <https://github.com/fedora-infra/fedbadges/blob/develop/README.rst>`_.

- For a diagram of the interacting pieces in the Fedora Badges system, see the `fedbadges diagrams <https://github.com/fedora-infra/fedbadges/blob/develop/diagrams/>`_.


******************
Pushing new badges
******************

Pushing badges consists of two operations:

#. Push badge assets (png, svg, yaml) to `fedora-badges`_ repository
#. Add badge to `badges.fedoraproject.org`_

Anyone with write permissions to `fedora-badges`_ can push badges.
Pull requests can also be used.
Only members of the ``sysadmin-badges`` FAS group *and* `admins of the web interface <add-tahrir-admins>`_ can add badges.

Badge artists and badge developers should push design assets (png and svg art) and rules (yaml) to the `fedora-badges`_ repository.

Preparing to create a badge
===========================

Once a badge is approved by the Design Team and has the `ready to push <https://pagure.io/fedora-badges/issues?status=Open&tags=ready+to+push>`_ tag, they are ready to be pushed.
Follow this checklist to push a new badge:

#. Ensure artwork is **approved** by Design Team
#. Ensure name and description of badge are clear
#. Ensure one of these requirements is met:
    - *Manually awarded badges*: What FAS accounts receive permissions to award badge
    - *Awarded via fedmsg rule*: YAML file with rules is present

If you are confused or something is missing, comment on the issue and remove the **ready to push** tag.

Add badge assets to Pagure
==========================

#. Download artwork and rule file (if applicable)
    - Double-check the artwork is actually the final version from ticket 
    - Open /view both art files to check they are not corrupt (in the past, corrupt images were accidentally pushed)
#. Give both art and rule files the **same name**, place into `pngs/ <https://pagure.io/fedora-badges/blob/master/f/pngs>`_, `svg/ <https://pagure.io/fedora-badges/blob/master/f/svgs>`_, `rules/ <https://pagure.io/fedora-badges/blob/master/f/rules>`_ directories
#. *On request only*: Generate a STL file for 3D-printing a badge
    - Change directories into ``bin/`` and run ``export.sh``.
      This creates a STL file for the badge and moves it to the correct place.
    - Check the ``README`` file in ``bin/`` for more info about the script.
#. Commit all files and push to `fedora-badges`_ (or make a PR)

Make sure files have a reasonable name (e.g. ``all-lowercase-dashes-only.png``).
Some types of badges follow a naming convention, like FAS group membership badges (i.e. ``fas-badge-name.png``) or Koji build badges (i.e. ``koji-badge-name.png``).
Follow past precedent where possible.
Later, the file name is used for the file on the front-end server (e.g. ``https://badges.fedoraproject.org/pngs/all-lowercase-dashes-only.png``).

Push assets to back-end server via Ansible
==========================================

*Note*: All assets must be in the ``master`` branch of `fedora-badges`_ before proceeding.
You must also be a member of ``sysadmin-badges`` in FAS for this to work.

Next, you need to move all assets from the Pagure repository to the back-end server.
This is done via an `Ansible playbook <https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/playbooks/manual/push-badges.yml>`_ from the ``batcave`` server.
Follow these steps to push the assets to production.

#. SSH into ``batcave01.phx2.fedoraproject.org``
#. Run the playbook: ``sudo rbac-playbook manual/push-badges.yml``
#. Enter FAS password and `2FA token <https://docs.pagure.org/infra-docs/sysadmin-guide/sops/2-factor.html>`_
#. Once finished, ensure badge is now publicly visible (i.e. ``https://badges.fedoraproject.org/pngs/<name>.png``)

Add automatically-awarded badges
================================

*Note*: Pay attention when following these steps.
It is easy to create a badge, but much harder to edit it later.
Double-check information is entered correctly in the YAML file on first go, or else you have to write SQL statements to fix it later.

Automatically awarded badges are created by the YAML rule file.
Once the Ansible playbook finishes, the badge is created and placed in the `badges index`_.
The only part *not* created automatically are tags (see `Badge metadata <badge-metadata>`_ below for info about tags).
To add new tags, find the badge on `badges.fedoraproject.org`_.
If you are logged in as an admin, there is a text field to enter in new tags.

Add manually-awarded badges
===========================

*Note*: Pay attention when following these steps.
It is easy to create a badge, but much harder to edit it later.
Double-check information is entered correctly on first go, or else you have to write SQL statements to fix it later.

Once the badge assets are pushed to the back-end, add the badge to the front-end, `Tahrir`.
The front-end is hosted at `badges.fedoraproject.org`_.
Follow these steps to add the badge in Tahrir.

#. Open *Admin* interface on `badges.fedoraproject.org`_
#. Go to the *Add badge* section.
#. Enter in all information as provided in the badge ticket (see `Badge metadata <badge-metadata>`_ below.)
#. Double-check all entered information is **correct and accurate**
#. Click *Create badge* to create new badge

The badge is now created.
You should be able to find in the `badges index`_.

Grant authorizations
--------------------

Manually-awarded badges require authorized users to issue a badge.
You can do this at the bottom of the *Admin* interface, near *Create Authorizations*.
Add the person to receive awarding privileges into the *Person Email* field.
**This must be formatted as their fedoraproject.org email address** (e.g. FASuser [at] fedoraproject [dot] org).
For badge name, use the slug (or badge name) from the URL of the badge (e.g. for `badges.fedoraproject.org/badge/commops-superstar <https://badges.fedoraproject.org/badge/commops-superstar>`_, this is ``commops-superstar``).

To add multiple users, repeat this process for each user.

.. _badge-metadata:

Badge metadata
==============

- **Name**: name of the badge – this determines URL of badge, so triple-check for typos
- **Image**: full link to the PNG (e.g. ``https://badges.fedoraproject.org/pngs/all-lowercase-dashes-only.png``)
- **Description**: badge description text (ensure there is *no* hanging whitespace)
- **Criteria**: link to the issue in `fedora-badges`_
- **Issuer**: keep the default
- **Tags**: comma-delimited list of tags
    - **Review other similar badges to ensure tags are correct.**
      Some tags are special and function as categories.
    - **Follow past precedent** for tags.
      Avoid creating new tags if at all possible.
    - Removing tags *is not* easy. Adding them later *is* easy.

Close out the ticket
====================

After pushing the badge, do some last checks to make sure the badge pushed correctly.
Make sure the page is viewable and double-check that it's categorized correctly in the `badges index`_.

Return to the Pagure issue for the badge.
Post a link to the pushed badge.
If you granted authorizations, list the FAS usernames you granted authorizations to.
After commenting, closing the issue as **pushed**.

Congratulations, you just pushed your very own Fedora Badge!


**********************
Manually award a badge
**********************

To perform this, you must be in the ``sysadmin-badges`` FAS group.

There is a script installed on ``badges-backend01`` in ``/usr/local/bin/award-badge``.
It has help options that you can pull up with ``award-badge -h``.
It takes a required ``--user FAS_USERNAME`` and a required ``--badge BADGE_ID`` option.
For example, the following invocation would award the "Associate Editor" badge to "ralph"::

  sudo /usr/local/bin/award-badge --user ralph --badge associate-editor

The ``BADGE_ID`` for a badge can be found by visiting its page on the web UI.
That badge can be found at ``https://badges.fedoraproject.org/badge/associate-editor``.

The ``award-badge`` script and source code is managed by ansible.git.
The source code is in `roles/badges/backend/files/award-badge <https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/roles/badges/backend/files/award-badge>`_.

Often enough, there is need for a workflow to batch award a badge to a number of people.
For instance, the *Keepin' Fedora Beautiful* badge comes from a member of the Design Team posting a ticket with a list of FAS usernames (i.e.
`fedora-badges#129 <https://pagure.io/fedora-badges/issue/129>`_).

For cases, like that you can ``wget`` the file with the list of FAS usernames on ``badges-backend01`` and run something like::

    $ for i in $(cat keepingbeautiful-list ) ; do
        sudo /usr/local/bin/award-badge --user $i --badge keepin-fedora-beautiful-f20;
    done


.. _manually-revoke-badge:

**********************************************
Manually revoke a badge or badge authorization
**********************************************

You may revoke badge or badge authorizations in a similar fashion to the ``award-badges`` script.
You may chain the invocation of the ``revoke-badge`` or ``revoke-authorization`` script in the same manner as the ``award-badges`` script.

Revoking a badge::

    sudo /usr/local/bin/revoke-badge --user ralph --badge associate-editor

Revoking an authorization::
    
    sudo /usr/local/bin/revoke-authorization --user ralph --badge associate-editor


.. _add-tahrir-admins:

***************************************
Add new admin to web interface (Tahrir)
***************************************

It would be nice if we could automatically grant admin access in the web interface to members of the ``sysadmin-badges`` FAS group.
We currently do not have this feature and must maintain the list of web UI admins separately.

The configuration file for the badges front-end web app is managed by ansible.git.
The source code is in `roles/badges/frontend/templates/tahrir.ini <https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/roles/badges/frontend/templates/tahrir.ini>`_.

In that file, find the ``tahrir.admin`` option.
It is a comma-separated list of email addresses that, when logged in, should be granted rights to access the admin panels at `badges.fedoraproject.org`_.

To add a new admin, add their ``FAS_USERNAME@fedoraproject.org`` email to that line, commit, and push.
Use Ansible to run the ``groups/badges-web.yml`` playbook to push the config change out to the web front-end nodes.


**********************************
Creating an invitation and QR code
**********************************

This is done through the admin panel of the web interface (although we can probably write a script for it to be used on the back-end node).

Invitations / QR codes are typically created for Fedora events.
For instance at the Flock 2013 Fedora Contributors conference, we created a badge to award attendees.
We followed the procedure below to generate an invitation and a QR code.
Next, the QR code was distributed to conference organizers.
They added the QR code to the program brochure that each attendee was given.
Then, any attendee that scanned the code was redirected on their phone to the badges app, where they were awarded the badge.

Create an invitation
====================

- Make sure you are an admin in the web interface and logged into `badges.fedoraproject.org`_
- Click the *Admin* link in the UI
- Under the *Invitations* section, add this information:

  - **Creation Date**: Optional.
    It defaults to the current date.
  - **Expiration Date**: Optional, but you probably want to specify one.
    It defaults to 2 hours from the current time.
    For instance, at the Flock 2013 conference, we set the expiration date at the end of the conference.
    Anyone who tried to claim the badge with the QR code after that time would be denied, with the message *this invitation is expired*.
  - **Badge ID**: "ID" of the badge you want to award.
    See the `section above <manually-revoke-badge>`_ for how to find a badge ID.
  - **Person email**: Email of a person in the badges database.
    In our case, use their Fedora email (e.g. ``FAS_USERNAME@fedoraproject.org``).

Now, the user you specified will have a link to the QR code and invite link on their profile page.
They can take initiative to distribute and share the badge as they wish.


******************************
Useful scripts for manual work
******************************

See ``ansible/roles/badges/backend/files/`` for the motherload.
These all get deployed to ``/usr/local/bin/`` on ``badges-backend01`` where you can login to execute them.

**edit-badge**
    Update the description and the criteria link for a badge (in the event that you created it incorrectly, or if feedback from other stakeholders requires us to change something)
**award-badge**
    Award a badge to a specific user.
**revoke-badge**
    Removes a badge from a user to whom it has been awarded erroneously.
    **Remember!**
    If you revoke a badge award from a user, you should also give them the ``consolation-prize`` badge as a token of apology.
**grant-authorization**
    Grant authorization rights on a badge to a privileged user.
    They can then create invitation links and QR codes for that badge as well as award it directly to other users from the web interface.
**revoke-authorization**
    Revoke those authorization rights for a user on a given badge.


.. _`badges index`: https://badges.fedoraproject.org/explore/badges
.. _`badges.fedoraproject.org`: https://badges.fedoraproject.org/
.. _fedbadges: https://github.com/fedora-infra/fedbadges
.. _`fedora-badges`: https://pagure.io/fedora-badges
.. _Tahrir: https://github.com/fedora-infra/tahrir
