.. title: OpenVPN SOP
.. slug: infra-openvpn
.. date: 2011-12-16
.. taxonomy: Contributors/Infrastructure

===========
OpenVPN SOP
===========

OpenVPN is our server->server VPN solution. It is deployed in a routeless
manner and uses ansible managed keys for authentication. All hosts should
be given static IP's and a hostname.vpn.fedoraproject.org DNS address.

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main

Location
	Phoenix

Servers
	bastion (vpn.fedoraproject.org)

Purpose
	Provides vpn solution for our infrastructure.

Add a new host
===============

Create/sign the keys
--------------------
From batcave01 check out the private repo::

   # This is to ensure that the clone is not world-readable at any point.
   RESTORE_UMASK=$(umask -p)
   umask 0077
   git clone /git/private
   $RESTORE_UMASK
   cd private/vpn/openvpn

Next prepare your environment and run the build-key script. This example
is for host "proxy4.fedora.phx.redhat.com"::

  . ./vars
  ./build-key $FQDN # ./revoke-full $FQDN to revoke keys that are no longer used.
  git add .
  git commit -a
  git push

Create Static IP
----------------

Giving static IP's out in openvpn is mostly painless. Take a look at other
examples but each host gets a file and 2 IP's.::

  git clone /git/ansible
  vi ansible/roles/openvpn/server/files/ccd/$FQDN

The file format should look like this::

  ifconfig-push 192.168.1.314 192.168.0.314

Basically the first IP is the IP that is contactable over the vpn and
should always take the format "192.168.1.x" and the PtPIP is the same ip
on a different network: "192.168.0.x"

Commit and install::

  git add .
  git commit -m "What have you done?"
  git push

And then push that out to bastion::

  sudo -i ansible-playbook $(pwd)/playbooks/groups/bastion.yml -t openvpn

Create DNS entry
----------------

After you have your static IP ready, just add the entry to DNS::

   git clone /git/dns && cd dns
   vi master/168.192.in-addr.arpa
   # pick out an ip that's unused
   vi master/vpn.fedoraproject.org
   git commit -m "What have you done?"
   ./do-domains
   git commit -m "done build."
   git push

And push that out to the name servers with::

   sudo -i ansible ns\* -a "/usr/local/bin/update-dns"

Update resolv.conf on the client
--------------------------------
To make sure traffic actually goes over the VPN, make sure the search line
in /etc/resolv.conf looks like::

  search vpn.fedoraproject.org fedoraproject.org

for external hosts and::

  search phx2.fedoraproject.org vpn.fedoraproject.org fedoraproject.org

for PHX2 hosts.

Remove a host
=============
::
  # This is to ensure that the clone is not world-readable at any point.
  RESTORE_UMASK=$(umask -p)
  umask 0077
  git clone /git/private
  $RESTORE_UMASK
  cd private/vpn/openvpn

Next prepare your environment and run the build-key script. This example
is for host "proxy4.fedora.phx.redhat.com"::

   . ./vars
   ./revoke-full $FQDN
   git add .
   git commit -a
   git push


TODO
====
Deploy an additional VPN server outside of PHX. OpenVPN does support
failover automatically so if configured properly, when the primary VPN
server goes down all hosts should connect to the next host in the list.
