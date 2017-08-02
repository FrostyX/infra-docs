.. title: MirrorManager Infrastucture SOP
.. slug: infra-mirrormanager
.. date: 2017-08-01
.. taxonomy: Contributors/Infrastructure

================================
MirrorManager Infrastructure SOP
================================

MirrorManager manages mirrors for fedora distribution.

.. contents::

Contact Information
===================

Owner
	 Fedora Infrastructure Team

Contact
	 #fedora-admin, sysadmin-main, sysadmin-web

Location
	 Phoenix

Servers
	 mm-frontend01, mm-frontend02, mm-frontend-checkin01, mm-backend01, mm-crawler01, mm-crawler02

Mirrorlist Servers
         Docker container on the proxy servers

Purpose
	 Manage mirrors for Fedora distribution

Description
===========

MirrorManager handles our mirroring system. It keeps track of lists of
valid mirrors and handles handing out metalink URLs to end users to download
packages from.

The backend server (*mm-backend01*) scans the master mirror (NFS mounted at /srv)
using the *mm2_update-master-directory-list* script (*umdl*) for changes. Changed
directories are detected by comparing the ctime to the value in the database.

The two crawlers (*mm-crawler01* and *mm-crawler02*) compare the content on the
mirrors with the results from *umdl* using RSYNC, HTTP, HTTPS. The crawler
process on *mm-crawler01* starts at 0:00 and 12:00 and at 2:00 and 14:00 on
*mm-crawler02*.

If the content on the mirrors is the same as on the master those mirrors are
included in the dynamic metalink/mirrorlist.

Every hour the backend server generates a python pickle which contains the
information about the state of each mirror. This pickle file is used by the
mirrorlist containers on the proxy servers to dynamically generate the
metalink/mirrorlist for each client individually.

The frontend servers (*mm-frontend01* and *mm-frontend02*) offer an interface
to manipulate the mirrors. Each mirror-admin can only change the details
of the associated mirror. Members of the FAS group *sysadmin-web* can seen
and change all existing mirrors.

The mirrorlist provided by the frontend servers has no actively consumed
content and is therefore heavily cached (12h). It is only used to give an
overview of existing mirrors.

Additionally the frontend servers provide:
 * an overview of the mirror list usage
   https://admin.fedoraproject.org/mirrormanager/statistics
 * a propagation overview
   https://admin.fedoraproject.org/mirrormanager/propgation
 * a mirror map
   https://admin.fedoraproject.org/mirrormanager/maps

The *mm-frontend-checkin01* server is only used for *report_mirror*
check-ins. This is used by mirrors to report their status independent
of the crawlers.

Release Preparation
===================

MirrorManager should automatically detect the new release version, and
will create a new Version() object in the database. This is visible on the
Version page in the web UI, and on https://admin.fedoraproject.org/mirrormanager/.

If the versioning scheme changes, it's possible this will fail. If so,
contact the Mirror Wrangler.

One Week After a Release
========================

In the first week after the release MirrorManager still uses the files
at fedora/linux/development/<version> and not at fedora/linux/releases/<version>

Once enough mirrors have picked up the files in the release directory
following script (on *mm-backend01*) can be used to change the paths
in MirrorManager::

  sudo -u mirrormanager mm2_move-devel-to-release --version=26 --category="Fedora Linux"
  sudo -u mirrormanager mm2_move-devel-to-release --version=26 --category="Fedora Secondary Arches"

Move to Archive
===============

Once the files of an EOL release have been copied to the archive directory
tree and enough mirrors have picked the files up at the archive location
there is also a script to adapt those paths in MirrorManager's database::

  sudo -u mirrormanager mm2_move-to-archive --originalCategory='Fedora EPEL' --directoryRe='/4/'

mirrorlist containers and mirrorlist servers
============================================
Every hour at :55 after the hour, mm-backend01 generates a pkl file with all the 
current mirrormanager infomration in it and syncs it to proxies and mirrorlist-servers. 
Each proxy accepts requests to mirrors.fedoraproject.org on apache, then uses haproxy
to determine what backend will reply. There are 2 containers defined on each proxy: 
mirrorlist1 and mirrorlist2. haproxy will look for those first, then fall back to any
of the mirrorlist servers defined over the vpn. 

At :15 after the hour, a script runs on all proxies:
/usr/local/bin/restart-mirrorlist-containers
This script starts up mirrorlist2 container, makes sure it can process requests and then
if so, restarts mirrorlist1 container with the new pkl data. If not, mirrorlist1 keeps 
running with the old data. During this process at least one (with mirrorlists servers 
as backup) server is processing requests so users see no issues. 

mirrorlist-containers log to /var/log/mirrormanager/mirrorlist{1|2}/ on the host 
proxy server. 

Troubleshooting and Resolution
==============================

Regenerating the Publiclist
---------------------------

On *mm-backend01*::

  sudo -u mirrormanager /usr/bin/mm2_update-mirrorlist-server
  sudo -u mirrormanager /usr/local/bin/sync_pkl_to_mirrorlists.sh

Those two commands generates a new mirrorlist pickle and transfers it to
the proxies.
The mirrorlist containers on the proxies are restarted 15 minutes after
each full hour.

The mirrorlist generation can take up to 20 minutes. If a faster solution
is required the mirrorlist pickle from the previous run is available at::

  /var/lib/mirrormanager/old/mirrorlist_cache.pkl

Updating the mirrorlist containers
----------------------------------

The container used for mirrorlists is the mirrormanager2-mirrorlist container
in Fedora dist git: https://src.fedoraproject.org/cgit/container/mirrormanager2-mirrorlist.git/
The one being used is defined in a ansible variable in:
roles/mirrormanager/mirrorlist_proxy/defaults/main.yml
and in turn used in systemd unit files for mirrorlist1 and mirrorlist2. 
To update the container used, update this variable, run the playbook 
and then restart the mirrorlist1 and mirrorlist2 containers on each proxy. 
Note that this may take a while the first time as the image has to be 
downloaded from our registiry. 

Debugging problems with mirrorlist container startup
----------------------------------------------------
Sometimes on boot some hosts won't be properly serving mirrorlists. 
This is due to a container startup issue. 
run: 'docker ps -a' as root to see the active containers. It will 
usually say something like 'exited(1)' or the like. Record the 
container id and then run: 'docker rm --force <containerid>'
then run 'docker ps -a' and confirm nothing shows. Then run
'systemctl start mirrorlist1' and it should correctly start 
mirrorlist1. 

General debugging for mirrorlist containers
-------------------------------------------
docker commands like 'docker ps -a' show a fair bit of information. 
Also, systemctl status mirrorlist1/2 or the journal should have information
when a container is failing. 

