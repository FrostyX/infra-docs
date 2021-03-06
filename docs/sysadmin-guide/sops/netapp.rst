.. title: Infrastructure Netapp SOP
.. slug: infra-netapp
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

=========================
Netapp Infrastructure SOP
=========================

Provides primary mirrors and additional storage in PHX2

Contents
========

1. Contact Information
2. Description
3. Public Mirrors

  1. Snapshots

4. PHX NFS Storage

  1. Access
  2. Snapshots

5. iscsi

  1. Updating LVM
  2. Mounting ISCSI

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main, releng

Location
	Phoenix, Tampa Bay, Raleigh

Servers
	batcave01, virt servers, application servers, builders, releng boxes

Purpose
	Provides primary mirrors and additional storage in PHX2

Description
===========

At present we have three netapps in our infrastructure. One in TPA, RDU
and PHX. For purposes of visualization its easiest to think of us as
having 4 netapps, 1 TPA, 1 RDU and 1 PHX for public mirrors. And an
additional 1 in PHX used for additional storage not related to the public
mirrors.

Public Mirrors
==============

The netapps are our primary public mirrors. The canonical location for the
mirrors is currently in PHX. From there it gets synced to RDU and TPA.

Snapshots
---------

Snapshots on the PHX netapp are taken hourly. Unfortunately the way it is
setup only Red Hat employees can access this mirror (this is scheduled to
change when PHX becomes the canonical location but that will take time to
setup and deploy). The snapshots are available, for example, on wallace in::

  /var/ftp/download.fedora.redhat.com/.snapshot/hourly.0

PHX NFS Storage
===============

There is a great deal of storage in PHX over NFS from the netapp there.
This storage includes the public mirror. The majority of this storage is
koji however there are a few gig worth of storage that goes to wiki
attachments and other storage needs we have in PHX.

You can access all of the nfs share shares at::

  batcave01:/mnt/fedora

or::

  ntap-fedora-a.storage.phx2.redhat.com:/vol/fedora/

Access
--------
   The netapp is provided by RHIS and as a result they also control access.
   Access is controlled by IP mostly and some machines have root squashed.
   Worst case scenario if batcave01 is not accessible, just bring another box
   up under its IP address and use that for an emergency.

Snapshots
---------
   
There are hourly and nightly snapshots on the netapp. They are available in::

  batcave01:/mnt/fedora/.snapshot

iscsi
=====
   
We have iscsi deployed in a number of locations in our infrastructure for
xen machines. To get a list of what xen machines are deployed with iscsi,
just run lvs::

  lvs /dev/xenGuests

Live migration is possible though not fully supported at this time. Please
shut a xen machine down and bring it up on another host. Memory is the
main issue here.

Updating LVM
-------------

iscsi is mounted all over the place and if one xen machine creates a
logical volume the other xen machines will have to pick up those changes.
To do this run::

 pvscan
 vgscan
 lvscan
 vgchange -a y

Mounting ISCSI
--------------

On reboots sometimes the iscsi share is not remounted. This should be
automated in the future but for now run::

  iscsiadm -m discovery -tst -p ntap-fedora-b.storage.phx2.redhat.com:3260
  sleep 1
  iscsiadm -m node -T iqn.1992-08.com.netapp:sn.118047036 -p 10.5.88.21:3260 -l
  sleep 1
  pvscan
  vgscan
  lvscan
  vgchange -a y

