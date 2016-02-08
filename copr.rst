.. title: Copr
.. slug: infra-copr
.. date: 2015-01-13
.. taxonomy: Contributors/Infrastructure
====
Copr
====

Copr is build system for 3rd party packages.

Frontend:
  - http://copr-fe.cloud.fedoraproject.org/
Backend:
  - http://copr-be.cloud.fedoraproject.org/
Package singer:
  - http://copr-keygen.cloud.fedoraproject.org/

Devel instances (NO NEED TO CARE ABOUT THEM, JUST THOSE ABOVE):
  - http://copr-fe-dev.cloud.fedoraproject.org/
  - http://copr-be-dev.cloud.fedoraproject.org/
  - http://copr-keygen-dev.cloud.fedoraproject.org/

Contact Information
====================
Owner
	 msuchy (mirek, vgologuz)
Contact
	 #fedora-admin, #fedora-buildsys
Location
	 Fedora Cloud
Purpose
	 Build system

TROUBLESHOOTING
================

Almost every problem with Copr is due problem in OpenStack, in such case::

  $ ssh root@copr-be.cloud.fedoraproject.org
  # copr-backend-service stop
  # source /home/copr/cloud/ec2rc.sh
  # /home/copr/delete-forgotten-instances.pl
  # # wait a minute and check
  # euca-describe-instances
  # # sometimes you have to run delete-forgotten-instances.pl as openstack is sometimes stuborn.
  #   copr-backend-service start

If this does not help you, then stop and kill all OpenStack VM builders and::

     $ ssh root@fed-cloud02.cloud.fedoraproject.org
     # source keystonerc
     # for i in $(nova-manage floating list | grep 7ed4d | grep None | sort | awk '{ print $2}')
        do nova-manage floating delete $i
        nova-manage floating create $i
       done
   
or even (USUALLY NOT NEEDED)::

  for i in /etc/init.d/openstack-*; do $i condrestart; done
 
and then start copr backend service again.

      # copr-backend-service stop

Sometimes OpenStack can not handle spawning too much VM at the same time.
So it is safer to edit on copr-be.cloud.fedoraproject.org::

      vi /etc/copr/copr-be.conf
 
and change::

      group0_max_workers=12
 
to "6". Start copr-backend service and some time later increase it to
original value. Copr automaticaly detect change in script and increase
number of workers.
     


Deploy information
==================

Using playbooks and rbac::

    $ sudo rbac-playbook groups/copr-backend.yml
    $ sudo rbac-playbook groups/copr-frontend.yml
    $ sudo rbac-playbook groups/copr-keygen.yml
    $ sudo rbac-playbook groups/copr-dist-git.yml

https://git.fedorahosted.org/cgit/copr.git/plain/copr-setup.txt

On backend should run copr-backend service (which spawns several processes).
Backend spawns VM from Fedora Cloud. You could not login to those machines directly.
You have to::

   $ ssh root@copr-be.cloud.fedoraproject.org
   # su - copr
   $ source /home/copr/cloud/ec2rc.sh
   $ euca-describe-instances
   # # instance type m1.builder are those spawned by backend, check 18th column with internal IP
   # # log there if you want
   $ ssh root@172.16.3.3
   # or terminate that instance (ID is in 2nd column)
   # euca-terminate-instances i-000003b3
   # #you can delete all instances in error state or simply forgotten by:
   # /home/copr/delete-forgotten-instances.pl

Logs
====
   
For backend
  /var/log/copr/backend.log /var/log/copr/workers/worker-*
   
For frontend:
  httpd logs: /var/log/httpd/{error,access}_log
 
For keygen:
  /var/log/copr-keygen/main.log

httpd logs: 
  /var/log/httpd/{error,access}_log

PPC64LE Builders
================

Builders for PPC64 are located at rh-power2.fit.vutbr.cz and anyone with access to buildsys ssh key can get there using keys as
  msuchy@rh-power2.fit.vutbr.cz

There are commands:
$ ls bin/
destroy-all.sh  reinit-vm26.sh  reinit-vm28.sh  virsh-destroy-vm26.sh  virsh-destroy-vm28.sh  virsh-start-vm26.sh  virsh-start-vm28.sh
get-one-vm.sh   reinit-vm27.sh  reinit-vm29.sh  virsh-destroy-vm27.sh  virsh-destroy-vm29.sh  virsh-start-vm27.sh  virsh-start-vm29.sh

bin/destroy-all.sh destroy all VM and reinit them
reinit-vmXX.sh  copy VM image from template
virsh-destroy-vmXX.sh  destroys VM
virsh-start-vmXX.sh starts VM
get-one-vm.sh  start one VM and return its IP - this is used in Copr playbooks.

In case of big queue of PPC64 tasks simply call bin/destroy-all.sh and it will destroy stuck VM and copr backend will spawn new VM.
