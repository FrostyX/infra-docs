.. title: Infrastucture libvirt tools SOP
.. slug: infra-libvirt
.. date: 2012-04-30
.. taxonomy: Contributors/Infrastructure

===================================
Fedora Infrastructure Libvirt Notes
===================================

Notes/FAQ on using libvirt/virsh/virt-manager in our environment

how do I migrate a guest from one virthost to another
=====================================================

multiple steps:
   
1. setup an unpassworded root ssh key to allow communication between 
    the two virthosts as root. This is only temporary, so, while scary
    it is not a big deal. Right now, this also means modifying 
    the ``/etc/ssh/sshd_config`` to ``permitroot without-password``.

2. Determine whatever changes need to be made to the guest. This can
   be the number of cpus, the amount of memory, or the disk location
   as this may not be standard on the current server. 

3. Make a dump of the current virtual guest using the virsh
    command. Use ``virsh dumpxml --migratable guestname`` and then edit
    any changes in disk layout, memory and cpu needed. 

4. setup storage on the destination end to match the source
   storage. ``lvs`` will give the amount of disk space. Due to some
   vaguries on disk sizes, it is always better to round up so if the
   original server says it is using 19.85 GB, make the next image 20
   GB. On the new server, use ``lvcreate -L+${SIZE}GB -n ${FQDN} vg_guests``
   
  
3. as root on source location::

        virsh -c qemu:///system  migrate --xml ${XML_FILE_FROM_3} \
          --copy-storage-all ${GUESTNAME} \
          qemu+ssh://root@destinationvirthost/system

      This should start the migration process and it will output absolutely 
      jack-squat on the cli for you to know this. On the destination system 
      go look in /var/log/libvirt/qemu/myguest.log (tail -f will show you the 
      progress results as a percentage completed)
    
4. Once the migration is complete you will probably need to run this
   on the new virthost::

     scp ${XML_FILE_FROM_3} root@destinationvirthost:

     ssh root@destinationvirthost
     virsh define ${XML_FILE_FROM_3}
     virsh autostart ${GUESTNAME}

5. Edit ansible host_vars of the guest and make sure that the
   associated values are correct::

     volgroup: /dev/vg_guests
     vmhost: virthost??.phx2.fedoraproject.org

6. Run the noc.yml ansible playbook to update nagios.

This should work for most systems. However in some cases, the virtual
servers on either side may have too much activity to 'settle' down
enough for a migration to work. In other cases the guest may be on a
disk like ISCSI which may not allow for direct migration. In this case
you will need to use a more direct movement.

1. Schedule outage time if any. This will need to be long enough to copy 
   the data from one host to another, so will depend on guest disk
   size. 

2. Turn off monitoring in nagios

3. setup an unpassworded root ssh key to allow communication between 
    the two virthosts as root. This is only temporary, so, while scary
    it is not a big deal. Right now, this also means modifying 
    the ``/etc/ssh/sshd_config`` to ``permitroot without-password``.

4. Determine whatever changes need to be made to the guest. This can
   be the number of cpus, the amount of memory, or the disk location
   as this may not be standard on the current server. 

5. Make a dump of the current virtual guest using the virsh
    command. Use ``virsh dumpxml --migratable guestname`` and then edit
    any changes in disk layout, memory and cpu needed. 

6. setup storage on the destination end to match the source
   storage. ``lvs`` will give the amount of disk space. Due to some
   vaguries on disk sizes, it is always better to round up so if the
   original server says it is using 19.85 GB, make the next image 20
   GB. On the new server, use ``lvcreate -L+${SIZE}GB -n ${FQDN} vg_guests``

7. Shutdown the guest.

8. Insert iptables rule for nc transfer::

     iptables -I INPUT 14 -s <source host> -m tcp -p tcp --dport 11111 -j ACCEPT

9. On the destination host: 

    -  RHEL-7::

        nc -l 11111 | dd of=/dev/<guest_vg>/<guest-partition>

10. On the source host::

	dd if=/dev/<guest_vg>/<guest-partition> | nc desthost 11111

    Wait for the copy to finish. You can do the following to track how
    far something has gone by finding the dd pid and then sending a
    'kill -USR1' to it.

11. Once the migration is complete you will probably need to run this
    on the new virthost::

     scp ${XML_FILE_FROM_3} root@destinationvirthost:

     ssh root@destinationvirthost
     virsh define ${XML_FILE_FROM_3}
     virsh autostart ${GUESTNAME}

12. Edit ansible host_vars of the guest and make sure that the
    associated values are correct::

     volgroup: /dev/vg_guests
     vmhost: virthost??.phx2.fedoraproject.org

13. Run the noc.yml ansible playbook to update nagios.
