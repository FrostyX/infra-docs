.. title: Bacula Infrastructure SOP
.. slug: infra-bacula
.. date: 2013-03-18
.. taxonomy: Contributors/Infrastructure

=========================
Bacula Infrastructure SOP
=========================

Bacula is our backup solution. Our backups run to backup01 and ultimately
to a tape robot. There is also a backup03 now with a newer bacula and tape
library. 

Contents
========

1. Contact Information
2. Troubleshooting and Resolution

  1. Intervention Required

3. Mysql insert failures
4. Restores/basic use
5. If/When bacula hangs
6. Rebooting the tape drive
7. common bacula commands

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main, sysadmin-backup
Location
	 Phoenix, ibiblio
Servers
	 backup01, backup03
Purpose
	 Our backup solution

Troubleshooting and Resolution
==============================

You can get to the bacula management console with::

  sudo /usr/sbin/bconsole

From there you can get a overview status with::

  status dir

Or a more specific status on the tape drive with::

  status 2 2

Intervention Required: Purging Tapes
====================================

If the tapes fill up, it may be required to purge the oldest volume. This
is done with the purge command. Select volume, then a list of volumes
comes up. Pick the oldest volume that has not already been purged or
recycled.::

  list volumes

then::

  purge

It will ask you for the media-id or volume -id. Enter the one from the
oldest tape listed above in list volumes.

After that the oldest tape will be purged, but bacula may not resume on it
yet. You need to load the tape to make sure it realizes it can go on::

  mount

It will ask for the device (say tape), then ask for the slot id. Enter the
number from above in list volumes.

Mysql insert failures
=====================

Sometimes backups will fail with a::

  Fatal error: Can't fill File table Query failed: INSERT INTO File
  (FileIndex, JobId, PathId, FilenameId, LStat, MD5)SELECT batch.FileIndex,
  batch.JobId, Path.PathId, Filename.FilenameId,batch.LStat, batch.MD5 FROM
  batch JOIN Path ON (batch.Path = Path.Path) JOIN Filename ON (batch.Name =
  Filename.Name): ERR=Lock wait timeout exceeded; try restarting transaction

This error is a issue with the old bacula and mysql we have, and there's
not much to be done about it. Start a new backup if one is needed or wait
for the next backup to complete.

Restores
========

When you need to restore files you'll need to find someone who can login to 
backup03.vpn.fedoraproject.org - it should be a member of sysadmin-main,
but they will also need their backup03 password which is NOT the same as 
the passwords on the other systems and ssh keys won't work.

Once logged in you will probably want to become root (sudo su - ) 
and start a ``screen`` session then run::

   /usr/sbin/bconsole

if you are doing a normal restore and you know the specific file(s) you want run::
  
   *restore
   *7
    (: Enter a list of files to restore)
   <enter my list of files>
   <enter on a blank line>
   <wait for it to restore the dirlisting - if it EVER does>
   say 'yes'
   *list jobs
   
   check the state of the RestoreFiles process.
   It should  restore the files to /tmp/bacula-restores/ on the remote system.


What to do when/if bacula hangs::

  service bacula-dir stop
  service bacula-sd stop
  service bacula-fd stop
  service mysqld restart
  service bacula-dir start
  service bacula-sd start
  service bacula-fd start

Try your action again in bconsole again :(


Rebooting the tape drive
========================
 
connect to tape02-mgmt.phx2.fedoraproject.org in your browser

login is: admin
password is the OLD fedora mgmt account password

click on ``operations -> system shutdown``
then select reboot from the page that comes up and submit.

the web interface will take a while to come back and then the tape library
will take betwen 4 and 10 minutes to be 'ready'. Just wait for it.

Sometimes after you do the above you either need to rescan thr scsi bus on 
backup03 or you need to reboot the system.




Bacula Cheat Sheet
==================

Cheat sheet:
  http://workaround.org/bacula-cheatsheet

*pasted here in case that site is down:*

Bacula is a nifty backup software that is network-capable and stores data
in the database for faster retrieval in case you need a certain file back.
As a big fan of cheat sheets I created this cheat sheet.

What's up?
----------

Which files shall be backed up?
  show filesets   I=Included, E=Excluded
What's the server doing?
  status dir
What's the status of a certain job?
  status jobid=xx
What's the client doing?
  status client
What's the streamer doing?
  status storage
Anything new?
  messages

Backing up
----------

Start a backup
  ``run``   ...and choose the backup job
Label a new tape 
  ``label`` ...and run mount afterwards

Restoring
---------

The common way (a user accidentally removed a file and wants the newest
version back from the tapes::

- Use the restore command.

- Choose option 5 (Select the most recent backup for a client).

- cd / ls / dir / mark / markdir / unmark / unmarkdir / lsmark /
  estimate / pwd / count / find

- done

See also:
http://bacula.org/rel-manual/Brief_Tutorial.html#SECTION000126000000000000000

Jobs
----

Last jobs
  ``list jobs``
  ...or ``list jobid=xx`` for a specific job

Statistics about last
  ``list jobtotal``
  
Which files were backed up?
  ``list files jobid=xx``

Job status
----------

Status means:

   T      Terminated normally
   C      Created but not yet running
   R      Running
   B      Blocked
   E      Terminated in Error
   e      Non-fatal error
   f      Fatal error
   D      Verify Differences
   A      Canceled by the user
   F      Waiting on the File daemon
   S      Waiting on the Storage daemon
   m      Waiting for a new Volume to be mounted
   M      Waiting for a Mount
   s      Waiting for Storage resource
   j      Waiting for Job resource
   c      Waiting for Client resource
   d      Wating for Maximum jobs
   t      Waiting for Start Time
   p      Waiting for higher priority job to finish

    

Tapes
-----

Which tapes are in the pool?
  ``list media``

Remove a tape
  ``delete media``

Which pools are defined?
  ``list pools``

Which tapes are/were used for a certain job?
  ``list jobmedia``

Assign a tape to a certain pool       
  ``add``

Change parameters of a tape
  ``update volume``

Troubleshooting
---------------

Erase a label on the tape 
  ``mt rewind && mt weof && mt rewind``


