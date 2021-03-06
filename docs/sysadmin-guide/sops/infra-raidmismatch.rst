.. title: Infrastructure Raid Mismatch Count SOP
.. slug: infra-raid-mismatch
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

======================================
Infrastructure/SOP/Raid Mismatch Count
======================================

What to do when a raid device has a mismatch count

Contents
========
1. Contact Information
2. Description
3. Correction

  1. Step 1
  2. Step 2

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main

Location
	All

Servers
	Physical hosts

Purpose
	Provides database connection to many of our apps.

Description
===========
In some situations a raid device may indicate there is a count mismatch as
listed in::

  /sys/block/mdX/md/mismatch_cnt

Anything other than 0 is considered not good. Though if the number is low
it's probably nothing to worry about. To correct this situation try the
directions below.

Correction
==========

More than anything these steps are to A) Verify there is no problem and B)
make the error go away. If step 1 and step 2 don't correct the problems,
PROCEED WITH CAUTION. The steps below, however, should be relatively safe.


Issue a repair (replace mdX with the questionable raid device)::

  echo repair > /sys/block/mdX/md/sync_action

Depending on the size of the array and disk speed this can take a while.
Watch the progress with::

  cat /proc/mdstat

Issue a check. It's this check that will reset the mismatch count if there
are no problems. Again replace mdX with your actual raid device.::

  echo check > /sys/block/mdX/md/sync_action

Just as before, you can watch the progress with::

  cat /proc/mdstat

