.. title: Infrastructure Kickstart SOP
.. slug: infra-kickstart
.. date: 2016-02-08
.. taxonomy: Contributors/Infrastructure

============================
Kickstart Infrastructure SOP
============================

Kickstart scripts provide our install infrastructure. We have a
plethora of different kickstarts to best match the system you are trying
to install. 

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Location
	 Everywhere we have machines. 
Servers
	 batcave01 (stores kickstarts and install media)
Purpose
	 Provides our install infrastructure

Introduction
============

Our kickstart infrastructure lives on batcave01. All
install media and kickstart scripts are located on batcave01. Because the
RHEL binaries are not public we have these bits blocked. You can add
needed IPs to (from batcave01)::

 ansible/roles/batcave/files/allows

Physical Machine (kvm virthost)
======================================

.. note:: PXE Booting

   If PXE booting just follow the prompt after doing the pxe boot (most hosts
   will pxeboot via console hitting f12).

Prep
----

This only works on an already booted box, many boxes at our colocations
may have to be rebuilt by the people in those locations first. Also make
sure the IP you are about to boot to install from is allowed to our IP
restricted infrastructure.fedoraproject.org as noted above (in
Introduction).

Download the vmlinuz and initrd images.

for a rhel6 install::

 wget http://infrastructure.fedoraproject.org/repo/rhel/RHEL6-x86_64/images/pxeboot/vmlinuz \
     -O /boot/vmlinuz-install
 wget http://infrastructure.fedoraproject.org/repo/rhel/RHEL6-x86_64/images/pxeboot/initrd.img \
     -O /boot/initrd-install.img

 grubby --add-kernel=/boot/vmlinuz-install \
        --args="ks=http://infrastructure.fedoraproject.org/repo/rhel/ks/hardware-rhel-6-nohd \
        repo=http://infrastructure.fedoraproject.org/repo/rhel/RHEL6-x86_64/ \
        ksdevice=link ip=$IP gateway=$GATEWAY netmask=$NETMASK dns=$DNS" \
        --title="install el6" --initrd=/boot/initrd-install.img

for a rhel7 install::

 wget http://infrastructure.fedoraproject.org/repo/rhel/RHEL7-x86_64/images/pxeboot/vmlinuz -O /boot/vmlinuz-install
 wget http://infrastructure.fedoraproject.org/repo/rhel/RHEL7-x86_64/images/pxeboot/initrd.img -O /boot/initrd-install.img

For phx2 hosts::

 grubby --add-kernel=/boot/vmlinuz-install \
        --args="ks=http://10.5.126.23/repo/rhel/ks/hardware-rhel-7-nohd \
        repo=http://10.5.126.23/repo/rhel/RHEL7-x86_64/ \
        net.ifnames=0 biosdevname=0 bridge=br0:eth0 bridge=br1:eth1 \
        ip={{ br0_ip }}::{{ gw }}:{{ nm }}:{{ hostname }}:br0:none" \
        --title="install el7" --initrd=/boot/initrd-install.img

For non phx2 hosts::

 grubby --add-kernel=/boot/vmlinuz-install \
        --args="ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-ext \
        repo=http://209.132.181.6/repo/rhel/RHEL7-x86_64/ \
        net.ifnames=0 biosdevname=0 bridge=br0:eth0 bridge=br1:eth1 \
        ip={{ br0_ip }}::{{ gw }}:{{ nm }}:{{ hostname }}:br0:none" \
        --title="install el7" --initrd=/boot/initrd-install.img

Fill in the br0 ip, gateway, etc

The default here is to use the hardware-rhel-7-nohd config which requires
you to connect via VNC to the box and configure its drives. If this is a
new machine or you are fine with blowing everything away, you can instead
use http://infrastructure.fedoraproject.org/rhel/ks/hardware-rhel-6-minimal
as your kickstart

If you know the number of hard drives the system has there are other
kickstarts which can be used. 

2 disk system::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-02disk
or external::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-02disk-ext

4 disk system::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-04disk
or external::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-04disk-ext

6 disk system::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-06disk
or external::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-06disk-ext

8 disk system::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-08disk
or external::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-08disk-ext
  
10 disk system::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-10disk
or external::
  ks=http://209.132.181.6/repo/rhel/ks/hardware-rhel-7-10disk-ext


Double and triple check your configuration settings (On RHEL-6 ``cat
/boot/grub/menu.lst`` and on RHEL-7 ``cat /boot/grub2/grub.cfg``),
especially your IP information. In places like ServerBeach not all hosts
have the same netmask or gateway. Once everything you are ready to run
the commands to get it set up to boot next boot.

RHEL-6::

 echo "savedefault --default=0 --once" | grub --batch
 shutdown -r now

RHEL-7::

  grub2-reboot 0
  shutdown -r now

Installation
------------

Once the box logs you out, start pinging the IP address. It will disappear
and come back. Once you can ping it again, try to open up a VNC session.
It can take a couple of minutes after the box is back up for it to
actually allow vnc sessions. The VNC password is in the kickstart script
on batcave01::

  grep vnc /mnt/fedora/app/fi-repo/rhel/ks/hardware-rhel-7-nohd

  vncviewer $IP:1

If using the standard kickstart script, one can watch as the install
completes itself, there should be no need to do anything. If using the
hardware-rhel-6-nohd script, one will need to configure the drives. The
password is in the kickstart file in the kickstart repo. 

Post Install
------------
Run ansible on the box asap to set root passwords and other security features. 
Don't leave a newly installed box sitting around.
