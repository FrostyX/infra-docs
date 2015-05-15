.. title: Infrastructure Machine Retirement SOP
.. slug: infra-machine-retirement
.. date: 2011-08-23
.. taxonomy: Contributors/Infrastructure

=================================
Infrastructure retire machine SOP
=================================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin
Location: 
  anywhere
Servers: 
  any
Purpose: 
  Makes sure decommisioning machines is correctly done

Introduction
============

When a machine (be it virtual instance or real physical hardware is
decommisioned, a set of steps must be followed to ensure that the machine
is properly removed from the set of machines we manage and doesn't cause
problems down the road.

Retire process
==============

1. Ensure that the machine is no longer used for anything. Use git-grep,
    stop services, etc.

2. Remove the machine from puppet. Make sure you not only remove the main
    machine name, but also any aliases it might have (or move them to an
    active server if they are active services. Make sure to search for the IP
    address(s) of the machine as well. Ensure dns is updated to remove the
    machine.

3. Remove the machine from any labels in hardware devices like consoles or
    the like.

4. Revoke the puppet cert for the machine.

5. Move the machine xml defintion to ensure it does NOT start on boot. You
    can move it to 'name-retired-YYYY-MM-DD'.

6. Ensure any backend storage the machine was using is freed or renamed to
    name-retired-YYYY-MM-DD

TODO
======
fill in commands

