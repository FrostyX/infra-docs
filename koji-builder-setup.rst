.. title: Infrastructure Koji Builder SOP
.. slug: infra-koji-builder
.. date: 2012-11-29
.. taxonomy: Contributors/Infrastructure

======================
Setup Koji Builder SOP
======================

Contents
========

- Setting up a new koji builder
- Resetting/installing an old koji builder

Builder Setup
==============
Setting up a new koji builder involves a goodly number of steps:

Network Overview
----------------

1. First get an instance spun up following the kickstart sop.

2. Define a hostname for it on the 125 network and a $hostname-nfs name
    for it on the .127 network.

3. make sure the instance has 2 network connections:

   - eth0 should be on the .125 network
   - eth1 should be on the .127 network

    For VM eth0 should be on br0, eth1 on br1 on the vmhost.

Setup Overview
--------------

- install the system as normal::

   virt-install -n $builder_fqdn -r $memsize \
   -f $path_to_lvm  --vcpus=$numprocs \
   -l http://10.5.126.23/repo/rhel/RHEL6-x86_64/ \
   -x "ksdevice=eth0 ks=http://10.5.126.23/repo/rhel/ks/kvm-rhel-6 \
    ip=$ip netmask=$netmask gateway=$gw dns=$dns \
    console=tty0 console=ttyS0" \
    --network=bridge=br0 --network=bridge=br1 \
    --vnc --noautoconsole

- run python ``/root/tmp/setup-nfs-network.py``
  this should print out the -nfs hostname that you made above

- change root pw

- disable selinux on the machine in /etc/sysconfig/selinux 

- reboot

- setup ssl cert into private/builders - use fqdn of host as DN
  
  - login to fas01 as root
  - ``cd /var/lib/fedora-ca``
  - ``./kojicerthelper.py normal --outdir=/tmp/  \
    --name=$fqdn_of_the_new_builder  --cadir=. --caname=Fedora``

  - info for the cert should be like this::

      Country Name (2 letter code) [US]:
      State or Province Name (full name) [North Carolina]:
      Locality Name (eg, city) [Raleigh]:
      Organization Name (eg, company) [Fedora Project]:
      Organizational Unit Name (eg, section) []:Fedora Builders
      Common Name (eg, your name or your servers hostname) []:$fqdn_of_new_builder
      Email Address []:buildsys@fedoraproject.org
  
  - scp the file in ``/tmp/${fqdn}_key_and_cert.pem`` over to lockbox01
  
  - put file in the private repo under ``private/builders/${fqdn}.pem``
  
  - ``git add`` + ``git commit``
  
  - ``git push``

- run puppet on the buildhost::
   
    /usr/sbin/puppetd --onetime --no-daemonize --verbose --color=false

- sign the puppet cert on lockbox::
   
    sudo puppetca -s $fqdn

- run ``./sync-hosts`` in infra-hosts repo; ``git commit; git push``

- as a koji admin run::

    koji add-host $fqdnr i386 x86_64
  
    (note: those are yum basearchs on the end - season to taste)

- run puppet again (and again and again) on the new builder


Resetting/installing an old koji builder
----------------------------------------

- disable the builder in koji (ask a koji admin)
- halt the old system (halt -p)
- undefine the vm instance on the buildvmhost::

    virsh undefine $builder_fqdn

- reinstall it - from the buildvmhost run::

   virt-install -n $builder_fqdn -r $memsize \
   -f $path_to_lvm  --vcpus=$numprocs \
    -l http://10.5.126.23/repo/rhel/RHEL6-x86_64/ \
    -x "ksdevice=eth0 ks=http://10.5.126.23/repo/rhel/ks/kvm-rhel-6 \
    ip=$ip netmask=$netmask gateway=$gw dns=$dns \
    console=tty0 console=ttyS0" \
    --network=bridge=br0 --network=bridge=br1 \
    --vnc --noautoconsole

- watch install via vnc::
  
    vncviewer -via bastion.fedoraproject.org $builder_fqdn:1

- when the install finishes:
  
  - start the instance on the buildvmhost::
    
      virsh start $builder_fqdn

  - set it to autostart on the buildvmhost::

     virsh autostart $builder_fqdn

- when the guest comes up
 
  - login via ssh using the temp root password
  - python /root/tmp/setup-nfs-network.py
  - change root password
  - disable selinux in /etc/sysconfig/selinux
  - reboot
  - on lockbox remove old puppet cert::
      
     puppetca --revoke --clean $builder_fqdn

  - on the builder run::
      
     /usr/sbin/puppetd --onetime --no-daemonize --verbose --color=false

  - sign the puppet cert on lockbox::
  
     puppetca -s $builder_fqdn

  - run puppet over and over until it stops changing things (1-4 times)

  - reboot one last time

  - ask a koji admin to re-enable the host


   

