.. title: Fedora Release Infrastructure SOP
.. slug: infra-releng
.. date: 2015-03-10
.. taxonomy: Contributors/Infrastructure

=================================
Fedora Release Infrastructure SOP
=================================

This SOP contains all of the steps required by the Fedora Infrastructure
team in order to get a release out. Much of this work overlaps with the
Release Engineering team (and at present share many of the same members).
Some work may get done by releng, some may get done by Infrastructure, as
long as it gets done, it doesn't matter.

Contact Information
===================

Owner: 
  Fedora Infrastructure Team, Fedora Release Engineering Team
Contact: 
  #fedora-admin, #fedora-releng, sysadmin-main, sysadmin-releng
Location: 
  N/A
Servers: 
  All
Purpose: 
  Releasing a new version of Fedora

Preparations
============

Before a release ships, the following items need to be completed. 

1. New website from the websites team (typically hosted at
   http://getfedora.org/_/)

2. Verify mirror space (for all test releases as well)

3. Verify with rel-eng permissions on content are right on the mirrors.  Don't leak.

4. Communication with Red Hat IS (Give at least 2 months notice, then
    reminders as the time comes near) (final release only)

5. Infrastructure change freeze

6. Modify Template:FedoraVersion to reference new version. (Final release only)

7. Move old releases to archive (post final release only)

8. Switch release from development/N to normal releases/N/ tree in mirror
    manager (post final release only)

Change Freeze
=============

The rules are simple:

* Hosts with the ansible variable "freezes" "True" are frozen. 

* You may make changes as normal on hosts that are not frozen. 
  (For example, staging is never frozen)

* Changes to frozen hosts requires a freeze break request sent to
  the fedora infrastructure list, containing a description of the 
  problem or issue, actions to be taken and (if possible) patches 
  to ansible that will be applied. These freeze breaks must then get
  two approvals from sysadmin-main or sysadmin-releng group members
  before being applied. 

* Changes to recover from outages are acceptable to frozen hosts if needed.

Change freezes will be sent to the fedora-infrastructure-list and begin 2
weeks before each release and the final release. The freeze will end one
day after the release. Note, if the release slips during a change freeze,
the freeze just extends until the day after a release ships.

You can get a list of frozen/non-frozen hosts by::

  git clone https://infrastructure.fedoraproject.org/infra/ansible.git
  scripts/freezelist -i inventory/inventory

Notes about release day
=======================

Release day is always an interesting and unique event. After the final
sprint from test to the final release a lot of the developers will be
looking forward to a bit of time away, as well as some sleep. Once Release
Engineering has built the final tree, and synced it to the mirrors it is
our job to make sure everything else (except the bit flip) gets done as
painlessly and easily as possible.

.. note:: All communication is typically done in #fedora-admin. Typically these
  channels are laid back and staying on topic isn't strictly enforced. On
  release day this is not true. We encourage people to come, stay in the
  room and be quiet unless they have a specific task or question releated to
  release day. Its nothing personal, but release day can get out of hand
  quick.

During normal load, our websites function as normal. This is especially
true since we've moved the wiki to mod_fcgi. On release day our load
spikes a great deal. During the Fedora 6 launch many services were offline
for hours. Some (like the docs) were off for days. A large part of this
outage was due to the wiki not being able to handle the load, part was a
lack of planning by the Infrastructure team, and part is still a mystery.
(There are questions as to whether or not all of the traffic was legit or
a ddos.

The Fedora 7 release went much better. Some services were offline for
minutes at a time but very little of it was out longer then that. The wiki
crashed, as it always does. We had made sure to make the fedoraproject.org
landing page static though. This helped a great deal though we did see
load on the proxy boxes as spiky.

Recent releases have been quite smooth due to a number of changes: we 
have a good deal more bandwith on master mirrors, more cpus and memory, 
as well as prerelease versions are much easier to come by for those
interested before release day. 

Day Prior to Release Day
========================

Step 1 (Torrent)
----------------
Setup the torrent. All files can be synced with the torrent box
but just not published to the world. Verify with sha1sum. Follow the
instructions on the torrentrelease.txt sop up to and including step 4.

Step 2 (Website)
----------------

Verify the website design / content has been finalized with the websites
team. Update the Fedora version number wiki template if this is a final
release. It will need to be changed in https://fedoraproject.org/wiki/Template:CurrentFedoraVersion

Additionally, there are redirects in the ansible 
playbooks/include/proxies-redirects.yml file for Cloud
Images. These should be pushed as soon as the content is available. 
See: https://fedorahosted.org/fedora-infrastructure/ticket/3866 for example

Step 3 (Mirrors)
----------------

Verify enough mirrors are setup and have Fedora ready for release. If for
some reason something is broken it needs to be fixed. Many of the mirrors
are running a check-in script. This lets us know who has Fedora without
having to scan everyone. Hide the Alpha, Beta, and Preview releases from
the publiclist page.

You can check this by looking at::

  wget "http://mirrors.fedoraproject.org/mirrorlist?path=pub/fedora/linux/releases/test/20-Alpha&country=global"

  (replace 20 and Alpha with the version and release.)

Release day
===========

Step 1 (Prep and wait)
----------------------

Verify the mirrors are ready and that the torrent has valid copies of its
files (use sha1sum)

Do not move on to step two until the Release Engineering team has given
the ok for the release. It is the releng team's decision as to whether or
not we release and they may pull the plug at any moment.

Step 2 (Torrent)
----------------

Once given the ok to release, the Infrastructure team should publish the
torrent and encourage people to seed. Complete the steps on the
http://infrastructure.fedoraproject.org/infra/docs/torrentrelease.txt
after step 4. 

Step 3 (Bit flip)
-----------------

The mirrors sit and wait for a single permissions bit to be altered so
that they show up to their services. The bit flip (done by the releng
team) will replicate out to the mirrors. Verify that the mirrors have
received the change by seeing if it is actually available, just use a spot
check. Once that is complete move on.

Step 4 (Taskotron) (final release only)
--------------------------------------- 

Please file a Taskotron ticket and ask for the new release support to be
added (log in to Phabricator using your FAS_account@fedoraproject.org email
address)
https://phab.qadevel.cloud.fedoraproject.org/maniphest/task/edit/form/default/?title=new%20Fedora%20release&priority=80&tags=libtaskotron

Step 5 (Website)
----------------

Once all of the distribution pieces are verified (mirrors and torrent),
all that is left is to publish the website. At present this is done by
making sure the master branch of fedora-web is pulled by the syncStatic.sh
script in ansible. It will sync in an hour normally but on release day 
people don't like to wait that long so do the following on sundries01

  sudo -u apache /usr/local/bin/lock-wrapper syncStatic 'sh -x /usr/local/bin/syncStatic'

Once that completes, on batcave01::

  sudo -i ansible proxy\* "/usr/bin/rsync --delete -a --no-owner --no-group bapp02::getfedora.org/ /srv/web/getfedora.org/"

Verify http://getfedora.org/ is working.

Step 6 (Docs)
-------------

Just as with the website, the docs site needs to be published. Just as
above follow the following steps::

  /root/bin/docs-sync

Step 7 (Monitor)
----------------

Once the website is live, keep an eye on various news sites for the
release announcement. Closely watch the load on all of the boxes, proxy,
application and otherwise. If something is getting overloaded, see
suggestions on this page in the "Juggling Resources" section.

Step 8 (Badges) (final release only)
------------------------------------

We have some badge rules that are dependent on which release of Fedora
we're on.  As you have time, please performs the following on your local
box::

  $ git clone ssh://git.fedorahosted.org/git/badges.git
  $ cd badges

Edit ``rules/tester-it-still-works.yml`` and update the release tag to match
the now old but stable release.  For instance, if we just released fc21,
then the tag in that badge rule should be fc20.

Edit ``rules/tester-you-can-pry-it-from-my-cold-dead-hands.yml`` and update
the release tag to match the release that is about to reach EOL.  For
instance, if we just released fc21, then the tag in that badge rule
should be fc19. Commit the changes::

  $ git commit -a -m 'Updated tester badge rule for f21 release.'
  $ git push origin master

Then, on batcave, perform the following::

  $ sudo -i ansible-playbook $(pwd)/playbooks/manual/push-badges.yml

Step 9 (Done)
--------------

Just chill, keep an eye on everything and make changes as needed. If you
can't keep a service up, try to redirect randomly to some of the mirrors.

Priorities
==========

Priorities of during release day (In order):

1. Website
    Anything related to a user landing at fedoraproject.org, and
    clicking through to a mirror or torrent to download something must be
    kept up. This is distribution, and without it we can potentially lose
    many users.

2. Linked addresses 
    We do not have direct control over what Digg,
    Slashdot or anyone else links to. If they link to something on the
    wiki and it is going down or link to any other site we control a
    rewrite should be put in place to direct them to
    http://fedoraproject.org/get-fedora.

3. Torrent 
    The torrent server has never had problems during a release.
    Make sure it is up.

4. Release Notes 
    Typically grouped with the docs site, the release
    notes are often linked to (this is fine, no need to redirect) but keep
    an eye on the logs and ensure that where we've said the release notes
    are, that they can be found there. In previous releases we sometimes
    had to make this available in more than one spot.

5. docs.fedoraproject.org 
    People will want to see whats new in Fedora
    and get further documentation about it. Much of this is in the release
    notes.

6. wiki 
    Because it is so resource heavy, and because it is so developer
    oriented we have no choice but to give the wiki a lower priority.

7. Everything else.

Juggling Resources
==================

In our environment we're running different things on many different
servers. Using Xen we can easily give machines more or less ram,
processors. We can take down builders and bring up application servers.
The trick is to be smart and make sure you understand what is causing the
problem. These are some tips to keep in mind:

* IPTables based bandwidth and connection limiting (successful in the
  past)

* Altering the weight on the proxy balancers

* Create static pages out of otherwise dynamic content

* Redirect pages to a mirror

* Add a server / remove un-needed servers

CHECKLISTS: 
===========

Alpha: 
------

* Announce infrastructure freeze 2 weeks before Alpha
* Change /topic in #fedora-admin
* mail infrastucture list a reminder. 
* File all tickets
* new website, check mirror permissions, mirrormanager, check
* mirror sizes, release day ticket. 

After release is a "go":

* Make sure torrents are setup and ready to go. 
* fedora-web needs a branch for fN-alpha. In it: 
  * Alpha used on get-prerelease
  * get-prerelease doesn't direct to release
  * verify is updated with Alpha info
  * releases.txt gets a branched entry for preupgrade
  * bfo gets updated to have a Alpha entry. 

After release:

* Update /topic in #fedora-admin
* post to infrastructure list that freeze is over. 

Beta: 
-----

* Announce infrastructure freeze 2 weeks before Beta
* Change /topic in #fedora-admin
* mail infrastucture list a reminder. 
* File all tickets
* new website 
* check mirror permissions, mirrormanager, check
  mirror sizes, release day ticket. 

After release is a "go":

* Make sure torrents are setup and ready to go. 
* fedora-web needs a branch for fN-beta. In it: 
* Beta used on get-prerelease
* get-prerelease doesn't direct to release
* verify is updated with Beta info
* releases.txt gets a branched entry for preupgrade
* bfo gets updated to have a Beta entry. 

After release:

* Update /topic in #fedora-admin
* post to infrastructure list that freeze is over. 

Final:
------

* Announce infrastructure freeze 2 weeks before Final
* Change /topic in #fedora-admin
* mail infrastucture list a reminder. 
* File all tickets
* new website, check mirror permissions, mirrormanager, check
* mirror sizes, release day ticket. 

After release is a "go":

* Make sure torrents are setup and ready to go. 
* fedora-web needs a branch for fN-alpha. In it: 
* get-prerelease does direct to release
* verify is updated with Final info
* bfo gets updated to have a Final entry. 
* update wiki version numbers and names. 

After release:

* Update /topic in #fedora-admin
* post to infrastructure list that freeze is over. 
* Move MirrorManager repository tags from the development/$version/
  Directory objects, to the releases/$version/ Directory objects. This is
  done using the ``move-devel-to-release --version=$version`` command on bapp02.
  This is usually done now a week or two after release. 
