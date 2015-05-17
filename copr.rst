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
  # systemctl stop copr-backend
  # source /home/copr/cloud/ec2rc.sh
  # /home/copr/delete-forgotten-instances.pl
  # # wait a minute and check
  # euca-describe-instances
  # # sometimes you have to run delete-forgotten-instances.pl as openstack is sometimes stuborn.
  #   systemctl start copr-backend

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

