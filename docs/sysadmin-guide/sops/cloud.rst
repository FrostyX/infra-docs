.. title: Fedora OpenStack Cloud 
.. slug: infra-openstack
.. date: 2015-04-28
.. taxonomy: Contributors/Infrastructure

================
Fedora OpenStack
================

Quick Start
===========

Controller::

  sudo  rbac-playbook hosts/fed-cloud09.cloud.fedoraproject.org.yml

Compute nodes::

  sudo  rbac-playbook groups/openstack-compute-nodes.yml

Description 
===========

If you need to install OpenStack install, either make sure the machine is clean.
Or use ``ansible.git/files/fedora-cloud/uninstall.sh`` script to brute force wipe off.

.. note::  by default, the script does not wipe LVM group with VM, you have to clean
  them manually. There is commented line in that script.

On fed-cloud09, remove the file ``/etc/packstack_sucessfully_finished`` to enforce run of packstack and few other commands.

After that wipe, you have to::

  ifdown eth1
  configure eth1 to become normal Ethernet with ip
  yum install openstack-neutron-openvswitch
  /usr/bin/systemctl restart neutron-ovs-cleanup
  ifup eth1

Additionally when reprovision OpenStack, all volumes on DellEqualogic are
preserved and you have to manually remove them (or remove them from OS before
it is reprovision). SSH to DellEqualogic (credentials are at the bottom of
``/etc/cinder/cinder.conf``) and run::

  show  (to get list of volumes)
  volume select <volume_name> offline
  volume delete <volume_name>

Before installing make sure:

  * make sure rdo repo is enabled
  * ``yum install openstack-packstack openstack-packstack-puppet openstack-puppet-modules``
  * ``vim /usr/lib/python2.7/site-packages/packstack/plugins/dashboard_500.py``
     and missing parentheses::

      ``host_resources.append((ssl_key, 'ssl_ps_server.key'))``

Now you can run playbook::

   sudo rbac-playbook hosts/fed-cloud09.cloud.fedoraproject.org.yml

If you run it after wipe (i.e. db has been reset), you have to:
 
  * import ssh keys of users (only possible via webUI - RHBZ 1128233
  * reset user passwords


Compute nodes
=============

Compute node is much easier and is written as role. Use::

  vars_files:
   - ... SNIP
   - /srv/web/infra/ansible/vars/fedora-cloud.yml
   - "{{ private }}/files/openstack/passwords.yml"

  roles:
  ... SNIP 
  - cloud_compute

Define a host variable in ``inventory/host_vars/FQDN.yml``::

  compute_private_ip: 172.23.0.10

You should also add IP to ``vars/fedora-cloud.yml``

And when adding new compute node, please update ``files/fedora-cloud/hosts``

.. important:: When reinstalling make sure you removed all members on Dell Equalogic
  (credentials are in /etc/cinder/cinder.conf on compute node) otherwise the
  space will be blocked!!!

Updates
=======
Our openstack cloud should have updates applied and reboots when the rest of our servers
are updated and rebooted. This will cause an outage, please make sure to schedule it. 

1. Stop copr-backend process on copr-be.cloud.fedoraproject.org
2. Kill all copr-builder instances.
3. Kill all transient/scratch instances. 
4. Update all instances we control. copr, persistent, infrastructure, qa etc. 
5. Shutdown all instances
6. Update and reboot fed-cloud09
7. Update and reboot all compute nodes
8. Start up all instances that are shutdown in step 5. 

TODO: add commands for above as we know them.

Troubleshooting 
===============

* could not connect to VM? - check your security group, default SG does not
   allow any connection.
* packstack end up with error, it is likely race condition in puppet - BZ 1135529. Just run it again.

* ERROR : append() takes exactly one argument (2 given
   ``vi /usr/lib/python2.7/site-packages/packstack/plugins/dashboard_500.py``
   and add one more surrounding () 

* Local ip for ovs agent must be set when tunneling is enabled 
   restart fed-cloud09 or:
   ssh to fed-cloud09; ifdown eth1; ifup eth1; ifup br-ex
 
* mongodb problem? follow
   https://ask.openstack.org/en/question/54015/mongodbpp-error-when-installing-rdo-on-centos-7/?answer=54076#post-id-54076

*  ``WARNING:keystoneclient.httpclient:Failed to retrieve management_url from token``::

    keystone --os-token $ADMIN_TOKEN --os-endpoint \
    https://fedorainfracloud.org:35357/v2.0/ endpoint-create --region 'RegionOne' \
    --service 91358b81b1aa40d998b3a28d0cfc86e7 --region 'RegionOne'  --publicurl \
    'https://fedorainfracloud.org:5000/v2.0'  --adminurl 'http://172.24.0.9:35357/v2.0' \
    --internalurl 'http://172.24.0.9:5000/v2.0' 

Fedora Classroom about our instance
===================================
http://meetbot.fedoraproject.org/fedora-classroom/2015-05-11/fedora-classroom.2015-05-11-15.02.log.html
