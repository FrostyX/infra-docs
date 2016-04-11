.. title: How to Archive Old Fedora Releases.
.. slug: archive-old-fedora
.. date: 2016-04-08 updated: 2016-04-08
.. taxonomy: Releng/Infrastructure

====================================
 How to Archive Old Fedora Releases
====================================

The Fedora download servers contain terabytes of data, and to allow
for mirrors to not have to take all of that data, infrastructure
regularly moves data of end of lifed releases (from /pub/fedora/linux)
to the archives section (/pub/archive/fedora/linux)

Steps Involved
==============

1. log into batcave01.phx2.fedoraproject.org and ssh to bodhi-backend01

2. Then change into the releases directory.

   cd /pub/fedora/linux/releases

4. Check to see that the target directory doesn't already exist.

   ls /pub/archive/fedora/linux/releases/

5. If the target directory does not already exist, do a recursive link
   copy of the tree you want to the target 

   cp -lvpnr 21 /pub/archive/fedora/linux/releases/21

6. If the target directory already exists, then we need to do a
   recursive rsync to update any changes in the trees since the
   previous copy. 

   rsync -avSHP --delete ./21/ /pub/archive/fedora/linux/releases/21/

7. We now do the updates and updates/testing in similar ways.

   cd ../updates/
   cp -lpnr 21 /pub/archive/fedora/linux/updates/21
   cd testing
   cp -lpnr 21 /pub/archive/fedora/linux/updates/testing/21

   cd ../updates/
   rsync -avSHP 21/ /pub/archive/fedora/linux/updates/21/
   cd testing
   rsync -avSHP 21/ /pub/archive/fedora/linux/updates/testing/21/

8. Announce to the mirror list this has been done and that in 2 weeks
   you will move the old trees to archives.

9. In two weeks, log into mirror manager

   
