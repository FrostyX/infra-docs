.. title: Torrent Releases Infrastructure SOP
.. slug: infra-torrent-releases
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure
.. highlight:: bash

===================================
Torrent Releases Infrastructure SOP
===================================

   http://torrent.fedoraproject.org/ is our master torrent server for
   Fedora distribution. It runs out of ServerBeach.

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-torrent group
Location
	 ibiblio
Servers
	 torrent.fedoraproject.org
Purpose
	 Provides the torrent master server for Fedora distribution

Torrent Release
===============

When you want to add a new torrent to the tracker at
[46]http://torrent.fedoraproject.org you need to take the following steps
to have it listed correctly:

1. login to torrent.fedoraproject.org. If you are unable to do so please
    contact the fedora infrastructure group about access. This procedure
    requires membership in the torrentadmin group.

2. Change the group ID to torrentadmin

  ::

    newgrp torrentadmin

3. Remove everything from the working directory /srv/torrent/new/fedora/

  ::

    rm -r /srv/torrent/new/fedora/*

4. rsync all the iso's from ibiblio

  ::

   rsync -avhHP rsync://download-ib01.fedoraproject.org/fedora-stage/<Version>_<Release>-<Label>/*/*/iso/ /srv/torrent/new/fedora/

5. Then cd into /srv/torrent/new/fedora/ to change the directory structure

  ::

    cd /srv/torrent/new/fedora/

6. The directories should be created by removing Label in the iso's name

  ::

    for iso in $(ls *iso); do dest=$(echo $iso|sed -e 's|-<Label>.iso||g' ); mkdir $dest; mv $iso $dest; done

7. Now copy the checksum's into the associated directories

  ::

    for checksum in $(ls *CHECKSUM); do for file in $(grep "SHA256 (" $checksum |sed -e 's|SHA256 (||g' -e 's|-<Label>.*||g' ); do cp $checksum $file ; done; done

8. Verify if all the checksums are copied into the right locations

  ::

    ls */

9. Remove the manifest files and checksums for netinst (since we dont mirror netinst images) and other files

  ::

    rm -rf *manifest *netinst* *CHECKSUM *i386 *x86_64

10. Run the maketorrent script from /srv/torrent/new/fedora/

  ::

    ../maketorrent *


  .. note::

    Next steps should be run 12 hours before the release time which is generally 14:00 UTC on Tuesday.

11. Grab fedora-torrent-init.py from releng scripts and change it to executable

  ::

    cd ~
    wget https://pagure.io/releng/raw/master/f/scripts/fedora-torrent-ini.py
    chmod 755 ~/fedora-torrent-ini.py

12. Run the following command from /srv/torrent/new/fedora/

  ::

    ~/fedora-torrent-ini.py <Version>_<Release> <Current_Date> > <Version>_<Release>.ini

13. Copy all the torrents to /srv/web/torrents/

  ::

    cp *torrent /srv/web/torrents/

14. Copy everything in /srv/torrent/new/fedora/ to /srv/torrent/btholding/

  ::

    cp -rl * /srv/torrent/btholding/

15. Copy the .ini file created in step 12 to /srv/torrent/torrent-generator/

  ::

    sudo cp <Version>_<Release>.ini /srv/torrent/torrent-generator/

16. Restart btseed and opentracker services

  ::

    systemctl restart btseed opentracker
 
  .. note::

    For final release, remove all the alpha and beta directories and torrent files corresponding to the
    release in /srv/torrent/btholding/ directory.


  .. note::

    At EOL of a release, remove all the directories and torrent files corresponding to the release in
    /srv/torrent/btholding/ directory.

