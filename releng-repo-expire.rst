.. title: Releng emergency repo expire
.. slug: infra-releng
.. date: 2016-01-09
.. taxonomy: Contributors/Infrastructure

============================
Releng emergency repo expire
=============================

Sometimes, we want to make sure that users pick up the very latest
version of the repo, and have them ignore any older version, at the
cost of lots of extra load on the master mirrors.
This SOP explains how a releng member can put this in motion.

Contents
========

1. Contact Information
2. Cautions
3. Preconditions
4. Steps

Contact Information
===================

Owner
	 Fedora Infrastructure Team

Contact
	 #fedora-admin, #fedora-releng

Persons
	 puiterwijk, dgilmore

Location
	 Phoenix

Servers
  - batcave01
  - mm-backend01

Purpose
     Expire old repo files.


Cautions
=========

Please note that because older versions are manually expired, clients will
only use the very latest version of the repo.
This does bring with it extra load on the master mirror until other mirrors
have picked up the latest mash.
Please make very sure you know what this encompasses, and notify the infra
team members so they are aware that extra load is to be expected!


Preconditions
=============

Before following this SOP, please make sure that:
1. The new updates repo is mashed and pushed to the master mirrors
2. The next UMDL run has occured to allow mirrormanager to pick up the new repo


Steps
=====

To expire old repo data:
1. Inform infrastructure team members in #fedora-noc about this so they are aware of
   extra load expected on the master mirrors.
2. Ssh into batcave01
3. Run: sudo -i rbac-playbook manual/releng-emergency-expire-old-repo.yml
4. Wait until this finishes. It will expire the old repo, create a new pickle, and sync
   the new pickle out, so it may take some time to run.
5. Profit: please use check_metalink[1] to verify that only the master mirror is in sync,
   but you should be done at this point.

[1]: https://github.com/puiterwijk/check_metalink
