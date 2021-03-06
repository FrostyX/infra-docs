.. title: Infrastructure iSCSI SOP
.. slug: infra-iscsi
.. date: 2011-08-23
.. taxonomy: Contributors/Infrastructure

=====
iSCSI
=====

iscsi allows one to share and mount block devices using the scsi protocol
over a network. Fedora currently connects to a netapp that has an iscsi
export.

Contents
========

1. Contact Information
2. Typical uses
3. iscsi basics

  1. Terms
  2. iscsi's basic login / logout procedure is

4. Loggin in
5. Logging out
6. Important note about creating new logical volumes

Contact Information
===================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-admin, sysadmin-main
Location
	Phoenix
Servers
	xen[1-15]
Purpose
	Provides iscsi connectivity to our netapp.

Typical uses
============

The best uses for Fedora are for servers that are not part of a farm or
live replicated. For example, we wouldn't put app1 on the iscsi share
because we don't gain anything from it. Shutting down app1 to move it
isn't an issue because app1 is part of our application server farm.

noc1, however, is not replicated. It's a stand alone box that, at best,
would have a non-live failover. By placing this host on an iscsi share, we
can make it more highly available as it allows us to move that box around
our virtualization infrastructure without rebooting it or even taking it
down.

iscsi basics
============

Terms
-------

* initiator means client
* target means server
* swab means mop
* deck means floor

iscsi's basic login / logout procedure is
-------------------------------------------
1. Notify your client that a new target is available (similar to editing
    /etc/fstab for a new nfs mount)
2. Login to the iscsi target (similar to running "mount /my/nfs"
3. Logout from the iscsi target (similar to running "umount /my/nfs"
4. Delete the target from the client (similar to removing the nfs mount
    from /etc/fstab)

Logging in
```````````
Most mounts are covered by ansible so this should be automatic. In the
event that something goes wrong though, the best way to fix this is:

- Notify the client of the target::

	iscsiadm --mode node --targetname iqn.1992-08.com.netapp:sn.118047036 --portal 10.5.88.21:3260 -o new

- Log in to the new target::

	iscsiadm --mode node --targetname iqn.1992-08.com.netapp:sn.118047036 --portal 10.5.88.21:3260 --login

- Scan and activate lvm::

	pvscan
	vgscan
	vgchange -ay xenGuests

Once this is done, one should be able to run "lvs" to see the logical
volumes

Logging out
```````````
Logging out isn't normally needed, for example rebooting a machine
automatically logs the initiator out. Should a problem arise though here
are the steps:

- Disable the logical volume::

	vgchange -an xenGuests

- log out::

	iscsiadm --mode node --targetname iqn.1992-08.com.netapp:sn.118047036 --portal 10.5.88.21:3260 --logout

.. note:: ``Cannot deactivate volume group``

  If the vgchange command fails with an error about not being able to
  deactivate the volume group, this means that one of the logical volumes is
  still in use. By running "lvs" you can get a list of volume groups. Look
  in the Attr column. There are 6 attrs listed. The 5th column usually has a
  '-' or an 'a'. 'a' means its active, - means it is not. To the right of
  that (the last column) you will see an '-' or an 'o'. If you see an 'o'
  that means that logical volume is still mounted and in use.

.. important:: Note about creating new logical volumes

    At present we do not have logical volume locking on the xen servers. This
    is dangerous and being worked on. Basically when you create a new volume
    on a host, you need to run::

	    pvscan
	    vgscan
	    lvscan

    on the other virtualization servers.
