.. title: Mass Upgrade Infrastructure SOP
.. slug: infra-mass-upgrade
.. date: 2013-07-29
.. taxonomy: Contributors/Infrastructure

===============================
Mass Upgrade Infrastructure SOP
===============================

Every once in a while, we need to apply mass upgrades to our servers for
various security and other upgrades.

Contents
========

1. Contact Information
2. Preparation
3. Staging
4. Special Considerations

    * Disable builders
    * Post reboot action
    * Schedule autoqa01 reboot
    * Bastion01 and Bastion02 and openvpn server
    * Special yum directives

5. Update Leader
6. Group A reboots
7. Group B reboots
8. Group C reboots
9. Doing the upgrade
10. Doing the reboot
11. Aftermath

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin, sysadmin-main,
  infrastructure@lists.fedoraproject.org, #fedora-noc
Location: 
  All over the world.
Servers: 
  all
Purpose: 
  Apply kernel/other upgrades to all of our servers

Preparation
===========

1. Determine which host group you are going to be doing updates/reboots
   on.

   Group "A" 
    servers that end users will see or note being down
    and anything that depends on them.
   Group "B" 
    servers that contributors will see or note being
    down and anything that depends on them.
   Group "C" 
    servers that infrastructure will notice are down,
    or are redundent enough to reboot some with others taking the
    load.

2. Appoint an 'Update Leader' for the updates.
3. Follow the [61]Outage Infrastructure SOP and send advance notification
   to the appropriate lists. Try to schedule the update at a time when
   many admins are around to help/watch for problems and when impact for
   the group affected is less. Do NOT do multiple groups on the same day
   if possible.
4. Plan an order for rebooting the machines considering two factors:

      * Location of systems on the kvm or xen hosts. [You will normally
        reboot all systems on a host together]
      * Impact of systems going down on other services, operations and
        users. Thus since the database servers and nfs servers are the
        backbone of many other systems, they and systems that are on the
        same xen boxes would be rebooted before other boxes.

5. To aid in organizing a mass upgrade/reboot with many people helping,
   it may help to create a checklist of machines in a gobby document.
6. Schedule downtime in nagios.
7. Make doubly sure that various app owners are aware of the reboots

Staging
=======
   Any updates that can be tested in staging or a pre-production environment
   should be tested there first. Including new kernels, updates to core
   database applications / libraries. Web applications, libraries, etc.

Special Considerations
======================

While this may not be a complete list, here are some special things that
must be taken into account before rebooting certain systems:

Disable builders
----------------

Before the following machines are rebooted, all koji builders should be
disabled and all running jobs allowed to complete:

  * db04
  * nfs01
  * kojipkgs02

Builders can be removed from koji, updated and re-added. Use::

 koji disable-host NAME

   and

 koji enable-host NAME

.. note:: you must be a koji admin

Additionally, rel-eng and builder boxes may need a special version of rpm. 
Make sure to check with rel-eng on any rpm upgrades for them. 

Post reboot action
------------------

The following machines require post-boot actions (mostly entering
passphrases). Make sure admins that have the passphrases are on hand for
the reboot:

 * backup-2 (LUKS passphrase on boot)
 * sign-vault01 (NSS passphrase for sigul service)
 * sign-bridge01 (NSS passphrase for sigul bridge service)
 * serverbeach* (requires fixing firewall rules): 

Each serverbeach host needs 3 or 4 iptables rules added anytime it's
rebooted or libvirt is upgraded:: 

  iptables -I FORWARD -o virbr0 -j ACCEPT 
  iptables -I FORWARD -i virbr0 -j ACCEPT 
  iptables -t nat -I POSTROUTING -s 192.168.122.3/32 -j SNAT --to-source 66.135.62.187

.. note:: The source is the internal guest ips, the to-source is the external ips that
          map to that guest ip. If there are multiple guests, each one needs 
          the above SNAT rule inserted. 

Schedule autoqa01 reboot
------------------------
There is currently an autoqa01.c host on cnode01. Check with QA folks
before rebooting this guest/host.

Bastion01 and Bastion02 and openvpn server
------------------------------------------

We need one of the bastion machines to be up to provide openvpn for all
machines. Before rebooting bastion02, modify:
``manifests/nodes/bastion0*.phx2.fedoraproject.org.pp`` files to start openvpn
server on bastion01, wait for all clients to re-connect, reboot bastion02
and then revert back to it as openvpn hub.

Special yum directives
----------------------

Sometimes we will wish to exclude or otherwise modify the yum.conf on a
machine. For this purpose, all machines have an include, making them read
[62]http://infrastructure.fedoraproject.org/infra/hosts/FQHN/yum.conf.include
from the infrastructure repo. If you need to make such changes, add them
to the infrastructure repo before doing updates.

Update Leader
=============

Each update should have a Leader appointed. This person will be in charge
of doing any read-write operations, and delegating to others to do tasks.
If you aren't specficially asked by the Leader to reboot or change
something, please don't. The Leader will assign out machine groups to
reboot, or ask specific people to look at machines that didn't come back
up from reboot or aren't working right after reboot. It's important to
avoid multiple people operating on a single machine in a read-write manner
and interfering with changes.

Group A reboots
===============

Group A machines are end user critical ones. Outages here should be
planned at least a week in advance and announced to the announce list.

List of machines currently in A group (note: this is going to be
automated)

These hosts are grouped based on the virt host they reside on:

* torrent02.fedoraproject.org
* ibiblio02.fedoraproject.org

* people03.fedoraproject.org
* ibiblio03.fedoraproject.org

* collab01.fedoraproject.org
* serverbeach09.fedoraproject.org

* db05.phx2.fedoraproject.org
* virthost03.phx2.fedoraproject.org

* db01.phx2.fedoraproject.org
* virthost04.phx2.fedoraproject.org

* db-fas01.phx2.fedoraproject.org
* proxy01.phx2.fedoraproject.org
* virthost05.phx2.fedoraproject.org

* ask01.phx2.fedoraproject.org
* virthost06.phx2.fedoraproject.org

These are the rest:

* bapp02.phx2.fedoraproject.org
* bastion02.phx2.fedoraproject.org
* app05.fedoraproject.org
* backup02.fedoraproject.org
* bastion01.phx2.fedoraproject.org
* fas01.phx2.fedoraproject.org
* fas02.phx2.fedoraproject.org
* log02.phx2.fedoraproject.org
* memcached03.phx2.fedoraproject.org
* noc01.phx2.fedoraproject.org
* ns02.fedoraproject.org
* ns04.phx2.fedoraproject.org
* proxy04.fedoraproject.org
* smtp-mm03.fedoraproject.org
* batcave02.phx2.fedoraproject.org
* mm3test.fedoraproject.org
* packages02.phx2.fedoraproject.org

Group B reboots
---------------
This Group contains machines that contributors use. Announcements of
outages here should be at least a week in advance and sent to the
devel-announce list.

These hosts are grouped based on the virt host they reside on:

* db04.phx2.fedoraproject.org
* bvirthost01.phx2.fedoraproject.org

* nfs01.phx2.fedoraproject.org
* bvirthost02.phx2.fedoraproject.org

* pkgs01.phx2.fedoraproject.org
* bvirthost03.phx2.fedoraproject.org

* kojipkgs02.phx2.fedoraproject.org
* bvirthost04.phx2.fedoraproject.org

These are the rest:

* koji04.phx2.fedoraproject.org
* releng03.phx2.fedoraproject.org
* releng04.phx2.fedoraproject.org

Group C reboots
---------------
Group C are machines that infrastructure uses, or can be rebooted in such
a way as to continue to provide services to others via multiple machines.
Outages here should be announced on the infrastructure list.

Group C hosts that have proxy servers on them:

* proxy02.fedoraproject.org
* ns05.fedoraproject.org
* hosted-lists01.fedoraproject.org
* internetx01.fedoraproject.org

* app01.dev.fedoraproject.org
* darkserver01.dev.fedoraproject.org
* fakefas01.fedoraproject.org
* proxy06.fedoraproject.org
* osuosl01.fedoraproject.org

* proxy07.fedoraproject.org
* bodhost01.fedoraproject.org

* proxy03.fedoraproject.org
* smtp-mm02.fedoraproject.org
* tummy01.fedoraproject.org

* app06.fedoraproject.org
* noc02.fedoraproject.org
* proxy05.fedoraproject.org
* smtp-mm01.fedoraproject.org
* telia01.fedoraproject.org

* app08.fedoraproject.org
* proxy08.fedoraproject.org
* coloamer01.fedoraproject.org

   Other Group C hosts:

* ask01.stg.phx2.fedoraproject.org
* app02.stg.phx2.fedoraproject.org
* proxy01.stg.phx2.fedoraproject.org
* releng01.stg.phx2.fedoraproject.org
* value01.stg.phx2.fedoraproject.org
* virthost13.phx2.fedoraproject.org

* db-fas01.stg.phx2.fedoraproject.org
* pkgs01.stg.phx2.fedoraproject.org
* packages01.stg.phx2.fedoraproject.org
* virthost11.phx2.fedoraproject.org

* app01.stg.phx2.fedoraproject.org
* koji01.stg.phx2.fedoraproject.org
* db02.stg.phx2.fedoraproject.org
* fas01.stg.phx2.fedoraproject.org
* virthost10.phx2.fedoraproject.org


* autoqa01.qa.fedoraproject.org
* autoqa-stg01.qa.fedoraproject.org
* bastion-comm01.qa.fedoraproject.org
* batcave-comm01.qa.fedoraproject.org
* virthost-comm01.qa.fedoraproject.org

* compose-x86-01.phx2.fedoraproject.org

* compose-x86-02.phx2.fedoraproject.org

* download01.phx2.fedoraproject.org
* download02.phx2.fedoraproject.org
* download03.phx2.fedoraproject.org
* download04.phx2.fedoraproject.org
* download05.phx2.fedoraproject.org

* download-rdu01.vpn.fedoraproject.org
* download-rdu02.vpn.fedoraproject.org
* download-rdu03.vpn.fedoraproject.org

* fas03.phx2.fedoraproject.org
* secondary01.phx2.fedoraproject.org
* memcached04.phx2.fedoraproject.org
* virthost01.phx2.fedoraproject.org

* app02.phx2.fedoraproject.org
* value03.phx2.fedoraproject.org
* virthost07.phx2.fedoraproject.org

* app03.phx2.fedoraproject.org
* value04.phx2.fedoraproject.org
* ns03.phx2.fedoraproject.org
* darkserver01.phx2.fedoraproject.org
* virthost08.phx2.fedoraproject.org

* app04.phx2.fedoraproject.org
* packages02.phx2.fedoraproject.org
* virthost09.phx2.fedoraproject.org

* hosted03.fedoraproject.org
* serverbeach06.fedoraproject.org

* hosted04.fedoraproject.org
* serverbeach07.fedoraproject.org

* collab02.fedoraproject.org
* serverbeach08.fedoraproject.org

* dhcp01.phx2.fedoraproject.org
* relepel01.phx2.fedoraproject.org
* sign-bridge02.phx2.fedoraproject.org
* koji03.phx2.fedoraproject.org
* bvirthost05.phx2.fedoraproject.org

* (disable each builder in turn, update and reenable).
* ppc11.phx2.fedoraproject.org
* ppc12.phx2.fedoraproject.org

* backup03

Doing the upgrade
=================
 
If possible, system upgrades should be done in advance of the reboot (with
relevant testing of new packages on staging). To do the upgrades, make
sure that the Infrastructure RHEL repo is updated as necessary to pull in
the new packages ([63]Infrastructure Yum Repo SOP)

On batcave01, as root run::

  func-yum [--host=hostname] update

..note: --host can be specified multiple times and takes wildcards.

pinging people as necessary if you are unsure about any packages.

Additionally you can see which machines still need rebooted with::

  sudo func-command --timeout=10 --oneline /usr/local/bin/needs-reboot.py | grep yes

You can also see which machines would need a reboot if updates were all
applied with::

  sudo func-command --timeout=10 --oneline /usr/local/bin/needs-reboot.py after-updates | grep yes

Doing the reboot
================
 
In the order determined above, reboots will usually be grouped by the
virtualization hosts that the servers are on. You can see the guests per
virt host on batcave01 in /var/log/virthost-lists.out

To reboot sets of boxes based on which virthost they are we've written a special
script which facilitates it::

   func-vhost-reboot virthost-fqdn

ex::

  sudo func-vhost-reboot virthost13.phx2.fedoraproject.org

Aftermath
=========

1. Make sure that everything's running fine
2. Reenable nagios notification as needed
3. Make sure to perform any manual post-boot setup (such as entering
    passphrases for encrypted volumes)
4. Close outage ticket.


Non virthost reboots:
---------------------

If you need to reboot specific hosts and make sure they recover - consider using::

  sudo func-host-reboot hostname hostname1 hostname2 ...

If you want to reboot the hosts one at a time waiting for each to come back before rebooting the next
pass a -o to func-host-reboot.



