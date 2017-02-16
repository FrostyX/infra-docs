.. title: Infrastructure 
.. slug: no-idea
.. date: 2015-07-09
.. taxonomy: Contributors/Infrastructure

==================================
Fedora Infrastructure Kpartx Notes
==================================

How to mount virtual partitions
===============================

There can be multiple reasons you need to work with the contents of a
virtual machine without that machine running.

1. You have decommisioned the system and found you need to get something
    that was not backed up.

2. The system is for some reason unbootable and you need to change some
    file to make it work.

3. Forensics work of some sort.

In the case of 1 and 2 the following commands and tools are
invaluable. In the case of 3, you should work with the Fedora Security
Team and follow their instructions completely.

Steps to Work With Virtual System
=================================

1. Find out what physical server the virtual machine image is on.

    A. Log into batcave01.phx2.fedoraproject.org

    B. search for the hostname in the file /var/log/virthost-lists.out::

        $ grep proxy01.phx2.fedoraproject.org /var/log/virthost-lists.out
        virthost05.phx2.fedoraproject.org:proxy01.phx2.fedoraproject.org:running:1

    C. If the image does not show up in the list then most likely it is
       an image which has been decommissioned. You will need to search
       the virtual hosts more directly::

	    # for i in `awk -F: '{print $1}' /var/log/virthost-lists.out |
                sort -u`; do
                    ansible $i -m shell -a 'lvs | grep proxy01.phx2'
                done

2. Log into the virtual server and make sure the image is shutdown. Even
   in cases where the system is not working correctly it may have still
   have a running qemu on the physical server. It is best to confirm that
   the box is dead.

          # virsh destroy <hostname>

3. We will be using the kpartx command to make the guest image ready for
   mounting. 

          # lvs | grep <hostname>
          # kpartx -l /dev/mapper/<volume>-<hostname>
          # kpartx -a /dev/mapper/<volume>-<hostname>
          # vgscan
          # vgchange -ay /dev/mapper/<new volume-name>
          # mount /dev/mapper/<partition we want> /mnt

4. Edit the files as needed.

5. Tear down the tree.
          
	  # umount /mnt
          # vgchange -an <volume-name>
          # vgscan
          # kpartx -d /dev/mapper/<volume>-<hostname>

