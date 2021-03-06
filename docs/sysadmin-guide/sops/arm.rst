.. title: Fedora ARM Infrastructure
.. slug: infra-arm
.. date: 2015-03-24
.. taxonomy: Contributors/Infrastructure

=========================
Fedora ARM Infrastructure
=========================

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main, sysadmin-releng
Location
	 Phoenix
Servers
	 arm01, arm02, arm03, arm04
Purpose
	 Information on working with the arm SOCs

Description
===========

We have 4 arm chassis in phx2, each containing 24 SOCs (System On Chip). 

Each chassis has 2 physical network connections going out from it. 
The first one is used for the management interface on each SOC. 
The second one is used for eth0 for each SOC.

Current allocations (2016-03-11): 

arm01
  one retrace instance, the rest primary builders attached to koji.fedoraproject.org
arm02 
  primary arch builders attached to koji.fedoraproject.org
arm03 
  In cloud network, public qa/packager and copr instances
arm04 
  primary arch builders attached to koji.fedoraproject.org

Hardware Configuration
=======================

Each SOC has:

* eth0 and eth1 (unused) and a management interface. 
* 4 cores
* 4GB ram
* a 300GB disk

SOCs are addressed by::

  arm{chassisnumber}-builder{number}.arm.fedoraproject.org

Where chassisnumber is 01 to 04 
and 
number is 00-23 

PXE installs
============
Kickstarts for the machines are in the kickstarts repo. 

PXE config is on noc01.  (or cloud-noc01.cloud.fedoraproject.org for arm03)

The kickstart installs the latests Fedora and sets them up with a base package set. 

IPMI tool Management
====================

The SOCs are managed via their mgmt interfaces using a custom ipmitool 
as well as a custom python script called 'cxmanage'. The ipmitool changes 
have been submitted upstream and cxmanage is under review in Fedora. 

The ipmitool is currently installed on noc01 and it has ability to 
talk to them on their management interface. noc01 also serves dhcp and
is a pxeboot server for the SOCs.

However you will need to add it to your path::

  export PATH=$PATH:/opt/calxeda/bin/

Some common commands: 

To set the SOC to boot the next time only with pxe::

  ipmitool -U admin -P thepassword -H arm03-builder11-mgmt.arm.fedoraproject.org chassis bootdev pxe

To set the SOC power off::

  ipmitool -U admin -P thepassword -H arm03-builder11-mgmt.arm.fedoraproject.org power off

To set the SOC power on::

  ipmitool -U admin -P thepassword -H arm03-builder11-mgmt.arm.fedoraproject.org power on

To get a serial over lan console from the SOC::

  ipmitool -U admin -P thepassword -H arm03-builder11-mgmt.arm.fedoraproject.org -I lanplus sol activate

DISK mapping
============

Each SOC has a disk. They are however mapped to the internal 00-23 in a non
direct manner:: 

  HDD Bay	EnergyCard	SOC (Port 1) SOC Num
  0		0	 	3    03
  1	 	0	 	0    00
  2		0	 	1    01
  3		0	 	2    02
  4		1	 	3    07
  5	 	1	 	0    04
  6	 	1	 	1    05
  7	 	1	 	2    06
  8	 	2	 	3    11
  9	 	2	 	0    08
  10	 	2		1    09
  11	 	2	 	2    10
  12	 	3	 	3    15
  13	 	3	 	0    12
  14	 	3	 	1    13
  15	 	3	 	2    14
  16	 	4	 	3    19
  17	 	4		0    16
  18	 	4	 	1    17
  19	 	4	 	2    18
  20	 	5	 	3    23
  21	 	5	 	0    20
  22	 	5	 	1    21
  23	 	5	 	2    22

Looking at the system from the front, the bay numbering starts from left to
right.

cxmanage
========

The cxmanage tool can be used to update firmware or gather diag info. 

Until cxmanage is packaged, you can use it from a python virtualenv::

  virtualenv --system-site-packages cxmanage
  cd cxmanage
  source bin/activate
  pip install --extra-index-url=http://sources.calxeda.com/python/packages/ cxmanage
  <use cxmanage>
  deactivate

Some cxmanage commands

::

  cxmanage sensor arm03-builder00-mgmt.arm.fedoraproject.org 
  Getting sensor readings...
  1 successes  |  0 errors  |  0 nodes left  |  .  

  MP Temp 0
  arm03-builder00-mgmt.arm.fedoraproject.org: 34.00 degrees C
  Minimum         : 34.00 degrees C
  Maximum         : 34.00 degrees C
  Average         : 34.00 degrees C
  ... (and about 20 more sensors)...

::

  cxmanage info arm03-builder00-mgmt.arm.fedoraproject.org 
  Getting info...
  1 successes  |  0 errors  |  0 nodes left  |  .  

  [ Info from arm03-builder00-mgmt.arm.fedoraproject.org ]
  Hardware version   : EnergyCard X04
  Firmware version   : ECX-1000-v2.1.5
  ECME version       : v0.10.2
  CDB version        : v0.10.2
  Stage2boot version : v1.1.3
  Bootlog version    : v0.10.2
  A9boot version     : v2012.10.16-3-g66a3bf3
  Uboot version      : v2013.01-rc1_cx_2013.01.17
  Ubootenv version   : v2013.01-rc1_cx_2013.01.17
  DTB version        : v3.7-4114-g34da2e2

firmware update::

  cxmanage --internal-tftp 10.5.126.41:6969 --all-nodes fwupdate package ECX-1000_update-v2.1.5.tar.gz arm03-builder00-mgmt.arm.fedoraproject.org

(note that this runs against the 00 management interface for the chassis and
updates all the nodes), and that we must run a tftpserver on port 6969 for 
firewall handling. 

Links 
======
http://sources.calxeda.com/python/packages/cxmanage/

Contacts 
=========
help.desk@boston.co.uk is the contact to send repair requests to. 
