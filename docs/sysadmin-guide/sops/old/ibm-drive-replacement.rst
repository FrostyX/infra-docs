.. title: Drive Replacement SOP
.. slug: infra-drive-replacement
.. date: 2012-07-13
.. taxonomy: Contributors/Infrastructure

====================================
Drive Replacement Infrastructure SOP
====================================

At present this SOP only works for the X series IBM servers.

We have multiple machines with lots of different drives in them. For the
most part now though, we are trying to standardise on IBM X series
servers. At present I've not figured out how to disable onboard raid, as a
result of this many of our servers have two raid 0 arrays then we do
software raid on this.

The system xen11 is currently an HP ProLiant DL180 G5 with its own
interesting RAID system (using Compaq Smart Array ccis). Like the IBM X
series each drive is considered a single RAID-0 instance which is then
accessed through a logical drive.

Contents
========

1. Contact Information
2. Verify the drive is dead

  1. Re-adding a drive (poor man's fix)

3. Actually replacing the drive (IBM)

  1. Collecting Data
  2. Call IBM
  3. Get the package, give access to the tech
  4. Prepwork before the tech arrives
  5. Tech on site
  6. Rebuild the array

4. Actually Replacing the Drive (HP)

  1. Collecting data
  2. Call HP
  3. Get the package, give access to the tech
  4. Prepwork before the tech arrives
  5. Tech on site
  6. Rebuild the array

5. Installing RaidMan (IBM Only)

Database - DriveReplacement

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Location
	 All
Servers
	 All
Purpose
	 Steps for drive replacement.

Verify the drive is dead
========================

::

  $ cat /proc/mdadm
  Personalities : [raid1]
   md0 : active raid1 sdb1[1] sda1[0]
   513984 blocks [2/2] [UU]

   md1 : active raid1 sdb2[2](F) sda2[0]
   487717248 blocks [2/1] [U_]

This indicates that md1 is in a degraded state and that /dev/sdb2 is the
failed drive. Notice that /dev/sdb1 (same physical drive as /dev/sdb2) is
not failed. /dev/md0 (not yet degraded) is showing a good state. This is
because /dev/md0 is /boot. If you run::

  touch /boot/t
  sync
  rm /boot/t

That should make /dev/md0 notice that its drive is also failed. If it does
not fail, its possible the drive is fine and that some blip happened that
caused it to get flagged as dead. It is also worthwhile to log in to
xenX-mgmt to determine if the RSAII adapter has noticed the drive is dead.

If you think the drive just had a blip and is fine, see "Re-adding" below

Re-adding a drive (poor man's fix)
-----------------------------------

Basically what we're doing here is making sure the drive is, infact, dead.
Obviously you don't want to do this more then once on a drive, if it
continues to fail. Replace it.

::

  # cat /proc/mdadm
  Personalities : [raid1]
  md0 : active raid1 sdb1[1] sda1[0]
       513984 blocks [2/2] [UU]

  md1 : active raid1 sdb2[2](F) sda2[0]
       487717248 blocks [2/1] [U_]
  # mdadm /dev/md1 --remove /dev/sdb2
  # mdadm /dev/md1 --add /dev/sdb2
  # cat /proc/mdstat
  md0 : active raid1 sdb1[1] sda1[0]
       513984 blocks [2/1] [U_]
         resync=DELAYED

   md1 : active raid1 sdb2[2] sda2[0]
         487717248 blocks [2/1] [U_]
         [=>...................]  recovery =  9.2% (45229120/487717248) finish=145.2min speed=50771K/sec

So we removed the bad drive, added it again and you can now see the
recovery status. Watch it carefully. If it fails again, time for a drive
replacement.

Actually replacing the drive (IBM)
==================================

Actually replacing the drive is a bit of a todo. If the box is in a RH
owned location, we'll have to file a ticket and get someone access to the
colo. If it is at another location, we may be able to just ship the drive
there and have someone do it on site. Please follow the below steps for
drive replacement.

Collecting Data
----------------

There's a not insignificant amount of data you'll need to place the call.
Please have the following information handy:

1) The hosts machine type (this is not model number).::

    # lshal | grep system.product
    system.product = 'IBM System x3550 -[7978AC1]-'  (string)

  In the above case, the machine type is encoded into [7978AC1]. And is just
  the first 4 numbers. So this machine type is 7978. M/T (machine type) is
  always 4 digits for IBM boxes.

2) Machine's serial number::

    # lshal | grep system.hardware.serial
    system.hardware.serial = 'FAAKKEE'  (string)

  The above's serial number is 'FAAKKEE'

3) Drive Stats

  There are two ways to get the drive stats. You can get some of this
  information via hal, but for the full complete information you need to
  either have someone physically go look at the drive (some of which is in
  inventory) or use RaidMan. See "Installing RaidMan" below for more
  information on how to install RaidMan.

  Specifically you need:

  - Drive Size (in G) 
  - Drive Type (SAS or SATA?) 
  - Drive Model 
  - Drive Vendor

  To get this information run::

    # cd /usr/RaidMan/
    # ./arcconf GETCONFIG 1

4) The phone number and address of the building where the drive is
    currently located. This will go to the RH cage.

    This information is located in the contacts.txt of private git repo on
    batcave01 (only available to sysadmin-main people)

  Call IBM

   Call 1-800-426-7378 and follow the directions they give you. You'll need
   to use the M/T above to get to the correct rep. They will ask you for the
   information above (you wrote it down, right?)

   When they agree to replace the drive, make sure to tell them you need the
   shipping number of the drive as well as the name of the tech who will do
   the drive replacement. Sometimes the tech will just bring the drive. If
   not though, you need to open a ticket with the colo to let them know a
   drive is coming.

  Get the package, give access to the tech

   As SOON as you get this information, open a ticket with RH. at
   is-ops-tickets redhat.com. Request a ticket ID from RH. If the tech has
   any issues getting into the colo, you can give the AT&T ticket request to
   the tech to get them in.

   NOTE: this can often take hours. We have 4 hour on site response time from
   IBM. This time goes very quickly, sometimes you may need to page out
   someone in IS to ensure it gets created quickly. To get this pager
   information see contacts.txt in batcave01's private repo (if batcave01 is down
   for some reason see the dr copy on backup2.fedoraproject.org:/srv/

  Prepwork before the tech arrives

   Really the big thing here is to remove the broken drive from the array. In
   our earlier example we found /dev/sdb failed. We'll want to remove it from
   both arrays:

 # mdadm /dev/md0 --remove /dev/sdb1
 # mdadm /dev/md1 --remove /dev/sdb2

   Next get the current state of the drives and save it somewhere. See
   "Installing RaidMan" for more information if RaidMan is not installed.

 # cd /usr/RaidMan
 # ./arcconf GETCONFIG 1 > /tmp/raid1.txt

   Copy /tmp/raid1.txt off to some other device and save it until the tech is
   on site. It should contain information about the failed drive.

  Tech on site

   When the tech is on site you may have to give him the rack location. All
   of our Mesa servers are in one location, "the same room that the desk is
   in". You may have to give him the serial number of the server, or possibly
   make it blink. It's either the first rack on the left labeled: "01 2 55"
   or "01 2 58".

   Once he's replaced the drive, he'll have you verify. Use the RaidMan tools
   to do the following:

 # cd /usr/RaidMan
 # ./arcconf RESCAN 1
 # ./arcconf GETCONFIG 1 > /tmp/raid2.txt
 # # arcconf CREATE <Controller#> LOGICALDRIVE [Options] <Size> <RAID#> <Channel# ID#>
 # ./arcconf create 1 LOGICALDRIVE 476790 Simple_volume 0 1

   First we're going to re-scan the array for the new drive. Then we'll
   re-get the configs. Compare /tmp/raid2.txt to /tmp/raid1.txt and verify
   the bad drive is fixed and that it has a different serial number. Also
   make sure its the correct size. Thank the tech and send him on his way.
   The last line there creates a new logical drive from the physical drive.
   "Simple_volume" tells it to create a raid0 array of one drive. The size
   was pulled out of our initial /tmp/raid1.txt (should match the other
   drive). The last two numbers are the Channel and ID of the new drive.

  Rebuild the array

   Now that the disk has been replaced we need to put a partition table on
   the new drive and add it to the array:

     * /dev/sdGOOD is the *GOOD* drive
     * /dev/sdBAD is the *BAD* drive

 # dd if=/dev/sdGOOD of=/tmp/sda-mbr.bin bs=512 count=1
 # dd if=/tmp/sda-mbr.bin of=/dev/sdBAD
 # partprobe

   Next re-add the drives to the array:

     * /dev/sdBAD1 and /dev/sdBAD2 are the partitons on the new drive which
       is no longer bad.

 # mdadm /dev/md0 --add /dev/sdBAD1
 # mdadm /dev/md1 --add /dev/sdBAD2
 # cat /proc/mdadm

   This starts rebuilding the arrays, the last line checks the status.

Actually Replacing the Drive (HP)

   Replacing the drive on the HP's is similar to the IBM's. First you will
   need to contact HP, then you will need to open a ticket with Red Hat's
   Helpdesk to get into the PHX2 facility. Then you will need to coordinate
   with the technician on the colocation's rules for entry and who to
   call/talk with.

  Collecting data

  Call HP

  Get the package, give access to the tech

  Prepwork before the tech arrives

  Tech on site

  Rebuild the array

   Now that the disk has been replaced we need to put a partition table on
   the new drive and add it to the array:

     * /dev/cciss/c0dGOOD is the *GOOD* drive. The HP utilities will have a
       code like 1I:1:1
     * /dev/cciss/c0dBAD is the *BAD* drive. The HP utilities will have a
       code like 2I:1:1

   First we need to create the logical drive on the system.

 # hpacucli controller serialnumber=P61630H9SVU4JF create type=ld sectors=63 drives=2I:1:1 raid=0

 # dd if=/dev/ccis/c0dGOOD of=/tmp/sda-mbr.bin bs=512 count=1
 # dd if=/tmp/sda-mbr.bin of=/dev/ccis/c0dBAD
 # partprobe

   Next re-add the drives to the array:

     * /dev/sdBAD1 and /dev/sdBAD2 are the partitons on the new drive which
       is no longer bad.

 # mdadm /dev/md0 --add /dev/sdBAD1
 # mdadm /dev/md1 --add /dev/sdBAD2
 # cat /proc/mdadm

   This starts rebuilding the arrays, the last line checks the status.

Installing RaidMan (IBM Only)

   Unfortunately there is no feasible alternative to managing IBM Raid Arrays
   without causing downtime. You can get and do this via the pre-POST
   interface. This requires downtime, and if the first drive is the failed
   drive, may result in a non-booting system. So for now RaidMan it is until
   we can figure out how to get rid of the raid controllers in these boxes
   completely.

 yum -y install compat-libstdc++-33.i686
 rpm -ihv https://infrastructure.fedoraproject.org/rhel/RaidMan/RaidMan-9.00.i386.rpm

   To verify installation has completed successfully:

 # cd /usr/RaidMan/
 # ./arcconf GETCONFIG 1

   This should print the current configuration of the raid controller and its
   logical drives.

