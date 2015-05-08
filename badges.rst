.. title: Badges SOP
.. slug: infra-badges
.. date: 2014-10-03
.. taxonomy: Contributors/Infrastructure

==========
badges SOP
==========

  Fedora Badges - a recognition system for contributors

Contact Information
-------------------

Owner:
  Badges SIG, Fedora Infrastructure Team, sysadmin-badges
Contact: 
  #fedora-apps, #fedora-admin, #fedora-noc
Servers:
  badges-web0*, badges-backend0*
Purpose:
  Award "badges" to Fedora Contributors

Description
-----------

The badge awarding backend daemon, fedbadges, wakes up when it receives a
fedmsg event. It compares that message and the history in datanommer against a
series of rules. If a contributor matches the criteria described in one of
those rules, then they are awarded a badge.

The frontend is a web application called tahrir.  It mostly is just an interface
for users to look at their badges, but it also has an admin interface for
manually adding new badges, awarding badges, and creating "invitations"
(qrcodes) for badges.

The badge awarding backend daemon, fedbadges, runs on the badges-backend01 node.
The process is "fedmsg-hub" and the logs are in /var/log/fedmsg/fedmsg-hub.log

The frontend runs under apache/mod_wsgi on badges-web0{1,2} nodes.

Pushing out new badges
----------------------

Badge artists and badge developers should be pushing stuff yaml rules and pngs
and svg art to this repo::

  https://git.fedorahosted.org/cgit/badges.git

There is a playbook that will take any new content from there and push it out
onto our servers.

    $ sudo -i ansible-playbook $(pwd)/playbooks/manual/push-badges.yml


Manually awarding a badge
-------------------------

To perform this, you must be in the sysadmin-badges FAS group.

There is a script installed on the badges-backend01 in
/usr/local/bin/award-badge.  It has help options that you can pull up with
"award-badge -h".  It takes a required --user FAS_USERNAME and a required
--badge BADGE_ID option.  For example, the following invocation would award the
"Associate Editor" badge to "ralph"::

    $ sudo /usr/local/bin/award-badge --user ralph --badge associate-editor

The BADGE_ID for a badge can be found by visiting its page on the web UI.  That
badge can be found at https://badges.fedoraproject.org/badge/associate-editor.

The award-badge script is managed by ansible and can be found in its git repo
in roles/badges-backend/files/award-badge on lockbox.

Often enough, there is need for a workflow to batch award a badge to a number of
people.  For instance, the "Keepin Fedora Beautiful" badge comes from a member
of the design team posting a ticket with a list of FAS usernames (i.e.,
https://fedorahosted.org/fedora-badges/ticket/129).

For cases, like that you can wget the file with the list of fas usernames on
badges-backend01 and run something like::

    $ for i in $(cat keepingbeautiful-list ) ; do
        sudo /usr/local/bin/award-badge --user $i --badge keepin-fedora-beautiful-f20;
    done

Adding a new admin to the web interface
---------------------------------------

It would be nice if we could automatically grant admin access in the web
interface to members of the sysadmin-badges fas group.  We currently do not
have this feature and must maintain the list of web ui admins separately.

The configuration file for the badges frontend webapp is managed by ansible and
can be found in the ansible git repo on lockbox in
``roles/badges-frontend/templates/tahrir.ini``

In that file, find the tahrir.admin option.  It is a comma-separated list of
email addresses that, when logged in, should be granted rights to access the
admin panels at https://badges.fedoraproject.org/

To add a new admin, add their ``FAS_USERNAME@fedoraproject.org`` email to that
line, commit and push.  Use ansible to run the ``groups/badges-web.yml`` playbook
to push the config change out to the web frontend nodes.

Creating an Invitation and QRCode
---------------------------------

This is done through the admin panel of the web interface (although we can
probably write a script for it to be used on the backend node).

Invitations/QRCodes are typically created for Fedora events.  For instance at
the Flock 2013 Fedora Contributors conference, we created a badge to be awarded
to the attendees.  We followed the procedure below to generate an invitation
and a qrcode and then distributed the qrcode to the conference organizers.
They added the qrcode to the pamphlet/brochure/program that each attendee was
given, and any attendee that scanned the qrcode was redirected on their phone
to the badges app where they were awarded the badge.

To create an invitation:

- Make sure you are an admin in the web interface and login at https://badges.fedoraproject.org/
- Click the 'Admin' link in the UI or navigate to https://badges.fedoraproject.org/admin
- Under the "Invitations" section, add this information:

  - "Creation Date" - this may be omitted.  It will default to the current date.
  - "Expiration Date" - this may be omitted.  It will default to 2 hours from
    the current time.  Typically you want to put something in for this.  For
    instance, at the Flock 2013 conference, we set the expiration date of the
    invitation to be at the end of the conference.  Anyone who tried to claim the
    badge with the given link or qrcode after that time would be denied with the
    message "this invitation is expired".
  - "Badge ID" - this should be the "id" of the badge you want to award.  See
    the section above on "manually awarding a badge" for how to find the id of a
    badge.
  - "Person ID" - the "id" of a person in the badges database -- this is very
    cumbersome.  Usually you only know their fas username, but here you need their
    actual ID number.  There is a script to retrieve is installed on
    badges-backend01.  Log in there and run::

        sudo /usr/local/bin/get-badges-person-id --user FASUSERNAME

    Use the id from that for this field.

Once you have filled out the fields above as so, the person you included in the
Person ID field will have a link to the qrcode and invite link on their profile
page at which point they can do whatever they want with it.
