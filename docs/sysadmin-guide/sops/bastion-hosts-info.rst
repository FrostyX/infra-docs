.. title: Bastion Hosts Info 
.. slug: infra-bastion
.. date: 2011-09-13
.. taxonomy: Contributors/Infrastructure

====================
Fedora Bastion Hosts
====================


Owner
  sysadmin-main
Contact
  admin@fedoraproject.org
Location
  phx2
Servers
  bastion01, bastion02, bastion-comm01
Purpose
  background and description of bastion hosts and their unique issues. 

Description
===========

There are 2 primary bastion hosts in the phx2 datacenter. One will be active 
at any given time and the second will be a hot spare, ready to take over. 
Switching between bastion hosts is currently a manual process that requires changes in ansible. 

There is also a bastion-comm01 bastion host for the qa.fedoraproject.org network. 
This is used in cases where users only need to access resources in that
qa.fedoraproject.org.

All of the bastion hosts have an external IP that is mapped into them. 
The reverse dns for these IPs is controlled by RHIT, so any changes must be 
carefully coordinated. 

The active bastion host performs the following functions: 

* Outgoing smtp from fedora servers. This includes email aliases, mailing list posts, 
  build and commit notices, mailing list posts, etc. 

* Incoming smtp from servers in phx2 or on the fedora vpn. Incoming mail directly 
  from the outside is NOT accepted or forwarded. 

* ssh access to all phx2/vpn connected servers. 

* openvpn hub. This is the hub that all vpn clients connect to and talk to each other via. 
  Taking down or stopping this service will be a major outage of services as all 
  proxy and app servers use the vpn to talk to each other. 

When rebuilding these machines, care must be taken to match up the dns names 
externally, and to preserve the ssh host keys.
