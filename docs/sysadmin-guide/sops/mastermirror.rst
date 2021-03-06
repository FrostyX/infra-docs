.. title: Master Mirror Infrastructure SOP
.. slug: infra-master-mirror
.. date: 2011-12-22
.. taxonomy: Contributors/Infrastructure

================================
Master Mirror Infrastructure SOP
================================

Contents                           
========                                      
 
1. Contact Information          
2. PHX Master Mirror Setup      
3. RDU I2 Master Mirror Setup   
4. Raising Issues               


Contact Information
===================

Owner: 
  Red Hat IS
Contact: 
  #fedora-admin, Red Hat ticket
Location: 
  PHX
Servers: 
  server[1-5].download.phx.redhat.com
Purpose: 
  Provides the master mirrors for Fedora distribution


PHX Master Mirror Setup
=======================

The master mirrors are accessible as::

     download1.fedora.redhat.com -> CNAME to download3.fedora.redhat.com
     download2.fedora.redhat.com -> currently no DNS entry
     download3.fedora.redhat.com -> 209.132.176.20
     download4.fedora.redhat.com -> 209.132.176.220
     download5.fedora.redhat.com -> 209.132.176.221
     
from the outside. download.fedora.redhat.com is a round robin to the above
 IPs.

The external IPs correspond to internal load balancer IPs that balance
between server[1-5]::

     209.132.176.20  -> 10.9.24.20
     209.132.176.220 -> 10.9.24.220
     209.132.176.221 -> 10.9.24.221

The load balancers then balance between the below Fedora IPs on the rsync
servers::

     10.8.24.21 (fedora1.download.phx.redhat.com) - server1.download.phx.redhat.com
     10.8.24.22 (fedora2.download.phx.redhat.com) - server2.download.phx.redhat.com
     10.8.24.23 (fedora3.download.phx.redhat.com) - server3.download.phx.redhat.com
     10.8.24.24 (fedora4.download.phx.redhat.com) - server4.download.phx.redhat.com
     10.8.24.25 (fedora5.download.phx.redhat.com) - server5.download.phx.redhat.com


RDU I2 Master Mirror Setup
==========================

.. note:: This section is awaiting confirmation from RH - information here may
   not be 100% accurate yet.

download-i2.fedora.redhat.com (rhm-i2.redhat.com) is a round robin
between::

     204.85.14.3 - 10.11.45.3
     204.85.14.5 - 10.11.45.5


Raising Issues
==============
   
Issues with any of this setup should be raised in a helpdesk ticket.
