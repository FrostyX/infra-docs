.. title: Infrastructure Host Rename SOP
.. slug: infra-host-rename
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

==============================
Infrastructure Host Rename SOP
==============================

This page is intended to guide you through the process of renaming a
virtual node.

Contents
========

1. Introduction
2. Finding out where the host is
3. Preparation
4. Renaming the Logical Volume
5. Doing the actual rename
6. Telling ansible about the new host
7. VPN Stuff

Introduction
============

Throughout this SOP, we will refer to the old hostname as $oldhostname and
the new hostname as $newhostname. We will refer to the Dom0 host that the
vm resides on as $vmhost.

If this process is being followed so that a temporary-named host can
replace a production host, please be sure to follow the [51]Infrastructure
retire machine SOP to properly decommission the old host before
continuing.

Finding out where the host is
=============================

In order to rename the host, you must have access to the Dom0 (host) on
which the virtual server resides. To find out which host that is, log in
to batcave01, and run::

  grep $oldhostname /var/log/virthost-lists.out

The first column of the output will be the Dom0 of the virtual node.

Preparation
===========

SSH to $oldhostname. If the new name is replacing a production box, change
the IP Address that it binds to, in ``/etc/sysconfig/network-scripts/ifcfg-eth0``.

Also change the hostname in ``/etc/sysconfig/network``.

At this point, you can ``sudo poweroff`` $oldhostname.

Open an ssh session to $vmhost, and make sure that the node is listed as
``shut off``. If it is not, you can force it off with::

  virsh destroy $oldhostname

Renaming the Logical Volume
============================
Find out the name of the logical volume (on $vmhost)::

  virsh dumpxml $oldhostname | grep 'source dev'

This will give you a line that looks like ``<source
dev='/dev/VolGroup00/$oldhostname'/>`` which tells you that
``/dev/VolGroup00/$oldhostname`` is the path to the logical volume.

Run ``/usr/sbin/lvrename`` (the path that you found above) (the path that you
found above, with $newhostname at the end instead of $oldhostname)`

For example::
  /usr/sbin/lvrename /dev/VolGroup00/noc03-tmp /dev/VolGroup00/noc01

Doing the actual rename
=======================
Now that the logical volume has been renamed, we can rename the host in
libvirt.

Dump the configuration of $oldhostname into an xml file, by running::

  virsh dumpxml $oldhostname > $newhostname.xml

Open up $newhostname.xml, and change all instances of $oldhostname to
$newhostname.

Save the file and run::

  virsh define $newhostname.xml

If there are no errors above, you can undefine $oldhostname::

  virsh undefine $oldhostname

Power on $newhostname, with::

  virsh start $newhostname

And remember to set it to autostart::

  virsh autostart $newhostname


VPN Stuff
=========

TODO
