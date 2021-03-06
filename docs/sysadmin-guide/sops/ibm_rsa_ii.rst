.. title: IBM RSA II Remote Management SOP
.. slug: infra-ibm-rsa-ii
.. date: 2011-08-23
.. taxonomy: Contributors/Infrastructure

=============================
IBM RSA II Infrastructure SOP
=============================

Many of our physical machines use RSA II cards for remote management.

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Location
	 PHX, ibiblio
Servers
	 All physical IBM machines
Purpose
	 Provide remote management for our physical IBM machines

Restarting the RSA II card
==========================

Normally, the RSA II can be restarted from the web/ssh interface. If you
are locked out of any outside access to the RSA II, follow these
instructions on the physical machine.

If the machine can be rebooted without issue, cut off all power to the
machine, wait a few seconds, and restart everything.

Otherwise, to restart the card without rebooting the machine:

1. Download and install the IBM Remote Supervisor Adapter II Daemon

    1. ``yum install usbutils libusb-devel`` # (needed by the RSA II daemon)
    
    2. Download the correct tarball from
        http://www-947.ibm.com/systems/support/supportsite.wss/docdisplay?lndocid=MIGR-5071676&brandind=5000008
        (TODO: check if this can be packaged in Fedora)
       
    3. Extract the tarball and run ``sudo ./install.sh --update``

2. Download and extract the IBM Advanced Settings Utility
    http://www-947.ibm.com/systems/support/supportsite.wss/docdisplay?lndocid=TOOL-ASU&brandind=5000016
          
          .. warning:: this tarball dumps files in the current working directory

3. Issue a ``sudo ./asu64 rebootrsa`` to reboot the RSA II.

4. Clean up: ``yum remove ibmusbasm64``

Other Resources
===============

http://www.redbooks.ibm.com/abstracts/sg246495.html may be a useful
resource to refer to when working with this.
