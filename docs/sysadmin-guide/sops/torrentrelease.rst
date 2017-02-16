.. title: Torrent Releases Infrastructure SOP
.. slug: infra-torrent-releases
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

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

2. upload the files you want to add to the torrent to
    torrent.fedoraproject.org:/srv/torrent/new/$yourOrg/

3. use sha1sum and verify the file you have uploaded matches the source

4. organize the files into subdirs (or not) as you would like

5. run /srv/torrent/new/maketorrent [file-or-dir-to-torrent]
    ([file-or-dir-to-torrent]) to generate a .torrent file or files

6. copy the .torrent file(s) to: /srv/torrent/www/torrents/$yourOrg/

7. cd to /srv/torrent/torrent-generator/ or /srv/torrent/spins-generator/
    (depending on if it is an official release or spins release)
 
8. add a .ini file in this directory for the content you'll be
    torrenting. If you're not doing a normal Fedora release the filename
    should in the brackets should be [$yourOrg/File-Of-1.1.torrent] — the
    format of each section should be as follows::
      
      [Zod-livecd-1-i386.torrent]
      description=Fedora Core 6 Zod LiveCD 1 iso image for i386.
      size=683M
      releasedate=2006-12-22
      group=Fedora Core 6 Zod LiveCD 1
 
9. mv all files from /srv/torrent/new/$yourOrg into
    /srv/torrent/btholding/ - this includes the files you uploaded as well
    as the .torrent files you've created.

Your files will be linked on the website and available on the tracker
after this.
