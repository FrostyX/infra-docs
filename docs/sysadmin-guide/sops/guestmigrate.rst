.. title: Guest Migration SOP
.. slug: infra-guest-migration
.. date: 2011-10-07
.. taxonomy: Contributors/Infrastructure

==============================
Guest migration between hosts. 
==============================

Move guests from one host to another. 

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main

Location
	PHX, Tummy, ibiblio, Telia, OSUOSL

Servers
	All xen servers, kvm/libvirt servers.

Purpose
	Migrate guests

How to do it
============

1. Schedule outage time if any. This will need to be long enough to copy 
   the data from one host to another, so will depend on guest disk
   size. 

2. Turn off monitoring in nagios

3. On new host create disk space for server::

      lvcreate -n app03 -L 32G vg_guests00

4. prepare old guest for migration:
   a) if system is xen, install a regular kernel
   b) look for entries for xenblk and hvc0 in /etc files

5. Shutdown the guest. 

6. ::
   
      virsh dumpxml guestname > guest.xml

7. Copy guest.xml to the new machine. You will need to make various
   edits depending on if the system was originally xen or such. I
   normally need to compare an existing xml on the target system and the
   one we dumped out to make up the differences.

8. Define the guest on the new machine: 'virsh define guest.xml'. 
   Depending on the changes in the xml this may not work and you will
   need to make many manual changes plus copy the guest.xml to
   ``/etc/libvirtd/qemu`` and do a ``/sbin/service libvirtd restart``

9. Insert iptables rule for nc transfer::

     iptables -I INPUT 14 -s <source host> -m tcp -p tcp --dport 11111 -j ACCEPT

10. On the destination host: 

    - RHEL-5::

        nc -l -p 11111 | dd of=/dev/mapper/<guest-partition>

    -  RHEL-6::

        nc -l 11111 | dd of=/dev/mapper/<guest-partition>

11. On the source host::

	    dd if=/dev/mapper/guest-partition | nc desthost 11111

    Wait for the copy to finish. You can do the following to track how
    far something has gone by finding the dd pid and then sending a
    'kill -USR1' to it.

11. start the guest on the new host:: 
    
      ``virsh start guest``

12. On the source host, rename storage and undefine guest so it's not started. 

