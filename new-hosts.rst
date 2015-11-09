.. title: Infrastucture DNS Host Addition SOP
.. slug: infra-dns-add
.. date: 2014-05-22
.. taxonomy: Contributors/Infrastructure

=====================
DNS Host Addition SOP
=====================

You should be able to follow these steps in order to create a new set of
hosts in infrastructure.

.. note::
  The DNS SOP at http://infrastructure.fedoraproject.org/infra/docs/dns.rst is outdated.

  Until it is updated, see the documentation here:
  http://infrastructure.fedoraproject.org/infra/dns/README

Walkthrough
===========

.. note::
  when the DNS SOP is updated, this should be added to it in an
  "Add a new host" section

Get a DNS repo checkout on batcave01
------------------------------------
::
  
  git clone /git/dns
  cd dns

An example always helps, so you can use git grep for something that has
been recently added to the data center/network that you want::
  
  git grep badges-web01
    built/126.5.10.in-addr.arpa:69       IN        PTR      badges-web01.stg.phx2.fedoraproject.org.
    [...lots of other stuff in built/ ignore these as they'll be generated later...]
    master/126.5.10.in-addr.arpa:69       IN        PTR      badges-web01.stg.phx2.fedoraproject.org.
    master/126.5.10.in-addr.arpa:101      IN        PTR      badges-web01.phx2.fedoraproject.org.
    master/126.5.10.in-addr.arpa:102      IN        PTR      badges-web02.phx2.fedoraproject.org.
    master/168.192.in-addr.arpa:109.1   IN      PTR     badges-web01.vpn.fedoraproject.org
    master/168.192.in-addr.arpa:110.1   IN      PTR     badges-web02.vpn.fedoraproject.org
    master/phx2.fedoraproject.org:badges-web01.stg   IN      A       10.5.126.69
    master/phx2.fedoraproject.org:badges-web01        IN  A       10.5.126.101
    master/phx2.fedoraproject.org:badges-web02        IN  A       10.5.126.102
    master/vpn.fedoraproject.org:badges-web01    IN A         192.168.1.109
    master/vpn.fedoraproject.org:badges-web02    IN A         192.168.1.110

So those are the files we need to edit.  In the above example, two of
those files are for the host on the PHX network.  The other two are for
the host to be able to talk over the VPN.  Although the VPN is not
always needed, the common case is that the host will need it.  (If any
clients *need to connect to it via the proxy servers* or it is not
hosted in PHX2 it will need a VPN connection).  An common exception is
here the staging environment: since we only have one proxy server in
staging and it is in PHX2, a VPN connection is not typically needed for
staging hosts.

Edit the zone file for the reverse lookup first (the \*in-addr.arpa file)
and find ips to use.  The ips will be listed with a domain name of
"unused."  If you're configuring a web application server, you probably
want two hosts for stg and at least two for production.  Two in
production means that we don't need downtime for reboots and updates.
Two in stg means that we'll be less likely to encounter problems related
to having multiple web application servers when we take a change tested
in stg into production::

  -105      IN        PTR      unused.
  -106      IN        PTR      unused.
  -107      IN        PTR      unused.
  -108      IN        PTR      unused.
  +105      IN        PTR      elections01.stg.phx2.fedoraproject.org.
  +106      IN        PTR      elections02.stg.phx2.fedoraproject.org.
  +107      IN        PTR      elections01.phx2.fedoraproject.org.
  +108      IN        PTR      elections02.phx2.fedoraproject.org.

Edit the forward domain (phx2.fedoraproject.org in our example) next::

  elections01.stg IN      A       10.5.126.105
  elections02.stg IN      A       10.5.126.106
  elections01     IN      A       10.5.126.107
  elections02     IN      A       10.5.126.108

Repeat these two steps if you need to make them available on the VPN.
Note: if your stg hosts are in PHX2, you don't need to configure VPM for
them as all our stg proxy servers are in PHX2.

Also remember to update the Serial at the top of all zone files.

Once the files are edited, you need to run a script to build the zones.
But first, commit the changes you just made to the "source"::

  git add .
  git commit -a -m 'Added staging and production elections hosts.'

Once that is committed, you need to run a script to build the zones and
then push them to the dns servers.::

  ./do-domains # This builds the files
  git add .
  git commit -a -m 'done build'
  git push

  $ sudo -i ansible ns\* -a '/usr/local/bin/update-dns' # This tells the dns servers to load the new files

Make certs 
==========

WARNING: If you already had a clone of private, make VERY sure to do a
git pull first! It's quite likely somebody else added a new host without
you noticing it, and you cannot merge the keys repos manually. (seriously,
don't: the index and serial files just wouldn't match up with the certificate,
and you would revoke the wrong certificate upon revocation).



When doing 2 factor auth for sudo, the hosts that we connect from need
to have valid SSL Certs.  These are currently stored in puppet::

  git clone /git/ansible-private && chmod 0700 ansible-private
  cd ansible-private/files/2fa-certs
  . ./vars
  ./build-and-sign-key $FQDN  # ex: elections01.stg.phx2.fedoraproject.org

The $FQDN should be the phx2 domain name if it's in phx2, vpn if not in
phx2, and if it has no vpn and is not in phx2 we should add it to the
vpn.::

  git add .
  git commit -a
  git push


NOTE: Make sure to re-run vars from the vpn repo. If you forget to do that,
You will just (try to) generate a second pair of 2fa certs, since the
./vars script create an environment var to the root key directory, which
is different.

Servers that are on the VPN also need certs for that. These are also stored
in puppet private::

  cd ansible-private/files/vpn/openvpn
  . ./vars
  ./build-and-sign-key $FQDN  # ex: elections01.phx2.fedoraproject.org
  ./build-and-sign-key $FQDN  # ex: elections02.phx2.fedoraproject.org

The $FQDN should be the phx2 domain name if it's in phx2, and just
fedoraproject.org if it's not in PHX2 (note that there is never .vpn
in the FQDN in the openvpn keys). Now commit and push.::

  git add .
  git commit -a
  git push


ansible
=======
::
  
  git clone /git/ansible
  cd ansible

To see an example::

  git grep badges-web01 (example)
  find . -name badges-web01\*
  find . -name badges-web'\'*'

inventory
---------

The ansible inventory file lists all the hosts that ansible knows about
and also allows you to create sets of hosts that you can refer to via a
group name.  For a typical web application server set of hosts we'd
create things like this::

  [elections]
  elections01.phx2.fedoraproject.org
  elections02.phx2.fedoraproject.org
  
  [elections-stg]
  elections01.stg.phx2.fedoraproject.org
  elections02.stg.phx2.fedoraproject.org

  [... find the staging group and add there: ...]

  [staging]
  db-fas01.stg.phx2.fedoraproject.org
  elections01.stg.phx2.fedoraproject.org
  electionst02.stg.phx2.fedoraproject.org

The hosts should use their fully qualified domain names here.  The rules
are slightly different than for 2fa certs.  If the host is in PHX2, use
the .phx2.fedoraproject.org domain name.  If they aren't in PHX2, then
they usually just have .fedoraproject.org as their domain name.  (If in
doubt about a not-in-PHX2 host, just ask).


VPN config
----------

If the machine is in VPN, create a file in ansible at
roles/openvpn/server/files/ccd/$FQDN with contents like:

  ifconfig-push 192.168.1.X 192.168.0.X

Where X is the last octet of the DNS IP address assigned to the host,
so for example for elections01.phx2.fedoraproject.org that would be:

  ifconfig-push 192.168.1.44 192.168.0.44


Work in progress 
================
From here to the end of file is still being worked on

host_vars and group_vars
------------------------

ansible consults files in inventory/group_vars and inventory/host_vars to set parameters that can be used in templates and playbooks.  You may need to edit these

It's usually easy to copy the host_vars and group_vars from an existing host that's similar to the one you are working on and then modify a few names to make it work.  For instance, for a web application server::

  cd ~/ansible/inventory/group_vars
  cp badges-web elections

Change the following::

  - fas_client_groups: sysadmin-noc,sysadmin-badges
  + fas_client_groups: sysadmin-noc,sysadmin-web

(You can change disk size, mem_size, number of cpus, and ports too if you need them).

Some things will definitely need to be defined differently for each host in a
group -- notably, ip_address.  You should use the ip_address you claimed in
the dns repo::

    cd ~/ansible/inventory/host_vars
    cp badges-web01.stg.phx2.fedoraproject.org elections01.stg.phx2.fedoraproject.org
    <edit appropriately>

The host will need vmhost declaration.  There is a script in
``ansible/scripts/vhost-info`` that will report how much free memory and how many
free cpus each vmhost has.  You can use that to inform your decision.
By convention, staging hosts go on virthost12.

Each vmhost has a different volume group.  To figure out what volume group that is,
execute the following command on the virthost.::

  vgdisplay

You mant want to run "lsblk" to check that the volume group you expect is the one
actually used for virtual guests.


.. note:: 
  | 19:16:01 <nirik> 3. add ./inventory/host_vars/FQDN host_vars for the new host.
  | 19:16:56 <nirik> that will have in it ip addresses, dns resolv.conf, ks url/repo, volume group to make the host lv in, etc etc.
  | 19:17:10 <nirik> 4. add any needed vars to inventory/group_vars/ for the group
  | 19:17:33 <nirik> this has memory size, lvm size, cpus, etc
  | 19:17:45 <nirik> 5. add tasks/virt_instance_create.yml task to top of group/host playbook
  | 19:18:10 <nirik> 6. run the playbook and it will go to the virthost you set, create the lv, guest, install it, wait for it to come up, then continue configuring it.

mailman.yml
  copy it from another file.

::

  ./ans-vhost-freemem --hosts=virtost\*


group vars

- vmhost (of the host that will host the VM)
- kickstart info (url of the kickstart itself and the repo)
- datacenter (although most likely won't change)

The host playbook is rather basic

- Change the name
- Most things won't change much

::

  ansible-playbook /srv/web/infra/ansible/infra/ansible/playbooks/grous/mailman.yml

References
==========

* The making a new instance section of: http://meetbot.fedoraproject.org/meetbot/fedora-meeting-1/2013-07-17/infrastructure-ansible-meetup.2013-07-17-19.00.html
