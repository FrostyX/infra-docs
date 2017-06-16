.. title: Guest Editing SOP
.. slug: infra-guest-editing
.. date: 2012-04-23
.. taxonomy: Contributors/Infrastructure

=================
Guest Editing SOP
=================

Various virsh commands

Contents
========

1. Contact Information
2. How to do it

    1. add/remove cpus
    2. resize memory

Contact Information
===================

Owner:
  Fedora Infrastructure Team
Contact:
  #fedora-admin, sysadmin-main
Location:
  PHX, Tummy, ibiblio, Telia, OSUOSL
Servers:
  All xen servers, kvm/libvirt servers.
Purpose:
  Resize guest disks

How to do it
============

Add cpu
-------

1. SSH to the virthost server

2. Calculate the number of CPUs the system needs

3. ``sudo virsh setvcpus  <guest> <num_of_cpus> --config`` - ie::

       sudo virsh setvcpus bapp01 16 --config

4. Shutdown the virtual system

5. Start the virtual system

6. Login and check that cpu count matches

Resize memory
-------------

1. SSH to the virthost server

2. Calculate the amount of memory the system needs in kb

3. ``sudo virsh setmem <guest> <num_in_kilobytes> --config`` - ie::

       sudo virsh setmem bapp01 16777216 --config

4. Shutdown the virtual system

5. Start the virtual system

6. Login and check that memory matches
