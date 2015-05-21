.. title: MirrorManager Infrastucture SOP
.. slug: infra-mirrormanager
.. date: 2012-04-24
.. taxonomy: Contributors/Infrastructure

================================
MirrorManager Infrastructure SOP
================================

Mirrormanager manages mirrors for fedora distribution.

Contents

1. Contact Information
2. Description

  1. Release Preparation

3. Troubleshooting and Resolution

  1. Regenerating the Publiclist
  2. Hung admin.fedoraproject.org/mirrormanager

Contact Information
===================

Owner
	 Fedora Infrastructure Team

Contact
	 #fedora-admin, sysadmin-main, sysadmin-web

Location
	 Phoenix

Servers
	 app01, app02, app03, app04, app05, app06, bapp02

Purpose
	 Manage mirrors for Fedora distribution

Description
===========

Mirrormanager handles our mirroring system. It keeps track of lists of
valid mirrors and handles handing out metalink urls to end users to download
packages from. 

On the backend app server (bapp01 or bapp02), mirrormanager runs crawlers to
check mirror contents, a job to update the public lists and other
housekeeping jobs. This data is then synced to the app* servers to serve to
end users. 

Release Preparation
===================

MirrorManager should automatically detect the new release version, and
will create a new Version() object in the database. This is visible on the
Version page in the web UI, and on mirrors.fp.o.

If the versioning scheme changes, it's possible this will fail. If so,
contact the Mirror Wrangler.

Troubleshooting and Resolution
==============================

Regenerating the Publiclist
---------------------------

On bapp02::

  sudo -u  mirrormanager -i

then::

  /usr/share/mirrormanager/server/update-mirrorlist-server > /tmp/mirrormanager-mirrorlist.log 2>&1 && \
  /usr/share/mirrormanager/mm_sync_out

To make this take effect immediately, you may need to remove the cache on
the proxies::

  # As root on proxy0[1-7]
  rm -rf /srv/cache/mod_cache/*

Hung admin.fedoraproject.org/mirrormanager 
------------------------------------------

This generally happens when an app server loses connection to db2.

1. on bapp02 and app[1-6], su up, and restart apache.

2. on bapp02, if crawlers and update-master-directory-list are likewise
    hung, kill them too. You may need to delete stale
    ``/var/lock/mirrormanager/*`` lockfiles as well.

Restarting mirrorlist_server 
----------------------------

mirrorlist_server on the app* machines is managed via supervisord. If you want
to restart it, use:: 

  supervisorctl restart


