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

2. setup storage on the destination end to match the source storage.
    If the path to the storage is not the same on both systems
    (ie: not the same path into ``/dev/Guests00/myguest``) then take a copy
    of the guest xml file from ``/etc/libvirt/qemu`` and modify it so it has 
    the right path. If you need to do this you need to add ``--xml thisfile.xml``
    to the arguments below AFTER the word 'migrate'
  
3. as root on source location::

        virsh -c qemu:///system  migrate --p2p --tunnelled \
          --copy-storage-all myguest \
          qemu+ssh://root@destinationvirthost/system

      This should start the migration process and it will output absolutely 
      jack-squat on the cli for you to know this. On the destination system 
      go look in /var/log/libvirt/qemu/myguest.log (tail -f will show you the 
      progress results as a percentage completed)
    
      .. note::  
          --p2p and --tunnelled are so it goes direct from one host to the other
          but uses ssh.
  
4. Once the migration is complete you will probably need to run this
   on the new virthost::

     virsh dumpxml myguest > /etc/libvirt/qemu/myguest.xml
     virsh destroy myguest
     virsh define /etc/libvirt/qemu/myguest.xml
     virsh autostart myguest
     virsh start myguest

