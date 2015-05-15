.. title: DNS Infrastructure SOP 
.. slug: infra-dns
.. date: 2013-09-02
.. taxonomy: Contributors/Infrastructure

================================
DNS repository for fedoraproject
================================

.. warning:: This is **out of date content**.  DNS is now managed in a
    different repo, /git/dns.  Please see the README file in that repo
    to manage DNS: http://infrastructure.fedoraproject.org/infra/dns/README

We've set this up so we can easily (and quickly) edit and deploy dns changes
with a record of who changed what and why. This system also lets us edit out
proxies from rotation for our many and varied websites quickly and with a
minimum of opportunity for error. Finally, it checks to make sure that all
of the zone changes will actually work before they are allowed.

DNS Infrastructure SOP
======================

We have 5 DNS servers:
	
ns01.fedoraproject.org
  hosted at Serverbeach
ns02.fedoraproject.org 
  hosted at ibiblio (ipv6 enabled)
ns03.phx2.fedoraproject.org 
  in phx2, internal to phx2. 
ns04.fedoraproject.org  
  in phx2, external.
ns05.fedoraproject.org 
  hosted at internetx

Contents
========
  
1. Contact Information
2. Troubleshooting, Resolution and Maintenance

  1. DNS update
  2. Adding a new zone

3. GeoDNS

  1. Non geodns fedoraproject.org IPs
  2. Adding and removing countries
  3. IP Country Mapping

4. resolv.conf

  1. Phoenix
  2. Non-Phoenix

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin, sysadmin-main, sysadmin-dns
Location: 
  ServerBeach and ibiblio and internetx and phx2. 
Servers: 
  ns01, ns02, ns03.phx2, ns04, ns05
Purpose: 
  Provides DNS to our users

Troubleshooting, Resolution and Maintenance


Editing the domain(s)
=====================

We have one domain which needs to be able to change on demand for proxy
rotation/removal - that's fedoraproject.org.

The other domains are edited only when we add/subtract a host or move it to
a new ip. Not much else.

If you need to edit a domain that is NOT fedoraproject.org:

- change to the 'master' subdir, edit the domain as usual
  (remember to  update the serial), save it.

If you need to edit fedoraproject.org:
 
- if you need to add/change a host in fedoraproject.org that is not '@' or
  'wildcard' then:
 
  - edit fedoraproject.org.template
  - make your changes
  - do not edit the serial or anything surrounded by {{  }} unless you
      REALLY know what you are doing.

- if you need to only add/remove a proxy during an outage or due to
    networking issue then run:

  - ``./zone-template fedoraproject.org.cfg disable ip [ip] [ip]``
      to disable the ip of the proxy you want removed.
  - ``./zone-template fedoraproject.org.cfg enable ip [ip] [ip]``
      reverses the disable
  - ``./zone-template fedoraproject.org.cfg reset``
      will reset to all ips enabled.

- if you want to add an all new proxy as '@' or 'wildcard' for
  fedoraproject.org:

  - edit fedoraproject.org.cfg
  - add the ip to the correct section of the ipv4 or ipv6 in the config.
  - save the file
  - check the file for validity by running: ``python fedoraproject.org.cfg``
    looking for errors or tracebacks.

In all cases then run:     

- ``./do-domains``

- if that completes successfully then run::

    git add .
    git commit -a
    git push
  
and then run this on all of the nameservers (as root)::

  /usr/local/bin/update-dns


To run this via ansible from lockbox do::

  sudo -i ansible ns\* -a "/usr/local/bin/update-dns"


this will pull from the git tree, update all of the zones and reload the
name server.



DNS update
==========

DNS config files are puppet managed on puppet1. The update is standard to
the puppet configs at http://fedoraproject.org/wiki/Infrastructure/Puppet/QuickStart

From puppet1::

  git clone /git/puppet
  cd puppet/modules/bind/files/master
  vi fedoraproject.org # Don't forget to increment the serial!
  cd ../..
  git commit -m "What you did"
  git push

It should update within a half hour. You can test the new configs with dig::

	dig @ns01.fedoraproject.org fedoraproject.org
	dig @ns02.fedorpaorject.org cvs.fedoraproject.org

Adding a new zone
=================

First name the zone and generate new set of keys for it. Run this on ns01.
Note it could take SEVERAL minutes to run::

  /usr/sbin/dnssec-keygen -a RSASHA1 -b 1024 -n ZONE c.fedoraproject.org 
  /usr/sbin/dnssec-keygen -a RSASHA1 -b 2048 -n ZONE -f KSK c.fedoraproject.org

Then copy the created .key and .private files to the private git repo (You
need to be sysadmin-main to do this). The directory is ``private/private/dnssec``.

- add the zone in zones.conf in ``puppet/modules/bind/files``
- save and commit - but do not push
- Add zone file to the master subdir in this repo
- git add and commit the file
- check the zone by running check-domains
- if you intend to have this be a dnssec signed zone then you must
  - create a new key::
      
      /usr/sbin/dnssec-keygen -a RSASHA1 -b 1024 -n ZONE $domain.org
      /usr/sbin/dnssec-keygen -a RSASHA1 -b 2048 -n ZONE -f KSK $domain.org
		
    - put the files this generates into /srv/privatekeys/dnssec on lockbox01
		- edit the do-domains file in this dir and your domain to the
		  signed_domains entry at the top
		- edit the zone you just created and add the contents of the .key files
		  to the bottom of the zone

If this is a subdomain of fedoraproject.org:

- run dnssec-dsfromkey on each of the .key files generated
- paste that output into the bottom of fedoraproject.org.template
- commit everything to the dns tree
- push your changes
- push your changes to the puppet repo
- test

If you add a new child zone, such as c.fedoraproject.org or
vpn.fedoraproject.org you will also need to add the contents of
dsset-childzone.fedoraproject.org (for example), to the main
fedoraproject.org zonefile, so that DNSSEC has a valid trust path to that
zone.
 
You also must set the NS delegation entries near the top of fedoraproject.org zone file
these are necessary to keep dnssec-signzone from whining with this error msg::
    
     dnssec-signzone: fatal: 'xxxxx.example.com': found DS RRset without NS RRset

Look for the: "vpn IN NS" records at the top of fedoraproject.org and copy them for the new child zone.
  

Editing/disabling a proxy in dns
================================

for more information on disabling a proxy from see the README file in the git dns tree
on lockbox or see http://infrastructure.fedoraproject.org/infra/dns/README


fedorahosted.org template
=========================
we want to create a separate entry for each fedorahosted project - but we
do not want to have to maintain it later. So we have a simple map that
let's us put the ones which are different in there and know where they
should go. The map's format is::

  projectname short_hostname-in-fedorahosted where it lives

examples::

	someproject git
	someproject svn
	someproject bzr
	someproject hosted-super-crazy

this will create cnames for each of them.

running ``./do-domains`` will take care of all that and update the serial
automatically.


GeoDNS
======

As part of our Content Distribution Network we use geodns for certain
zones. At the moment just ``fedoraproject.org`` and ``*.fedoraproject.org`` zones.
We've got proxy servers all over the US, in Europe and in the UK. We are
now sending users to proxy servers that are near them. The current list of
available 'zone areas' are:

* DEFAULT
* EU
* GB
* NA

DEFAULT contains all the zones. So someone who does not seem to be in or
near the EU, GB, or NA would get directed to any random set. (South Africa
for example doesn't get directed to any particular server).

.. important::
   Don't forget to increase the serial number in the fedoraproject.org zone
   file. Even if you're making a change to one of the geodns IPs. There is
   only one serial number for all setups and that serial number is in the
   fedoraproject.org zone.

.. note:: Non geodns fedoraproject.org IPs
  If you're adding as server that is just in one location, and isn't going
  to get geodns balanced. Just add that host to the fedoraproject.org zone.

Adding and removing countries
-----------------------------

Our setup actually requires us to specify which countries go to which
servers. To do this, simply edit the named.conf file in puppet. Below is
an example of what counts as "NA" (North America).::

  view "NA" {
         match-clients { US; CA; MX; };
         recursion no;
         zone "fedoraproject.org" {
                 type master;
                 file "master/NA/fedoraproject.org.signed";
         };
         include "etc/zones.conf";
  };

IP Country Mapping
------------------

The IP -> Location mapping is done via a config file that exists on the
dns servers themselves (it's not puppet controlled). The file, located at
``/var/named/chroot/etc/GeoIP.acl`` is generated by the ``GeoIP.sh`` script
(that script is in puppet).

.. warning:: 
  This is known to be a less efficient means of doing geodns then the
  patched version from kernel.org. We're using this version at the moment
  because it's in Fedora and works. The level of DNS traffic we see is
  generally low enough that the inefficiencies aren't that noticed. For
  example, average load on the servers before this geodns was .2, now it's
  around .4

resolv.conf
===========

In order to make the network more transparent to the admins we do a lot of
search based relative names. Below is a list of what a resolv.conf should
look like.

.. important:: 
  Any machine that is not on our vpn or has not yet joined the vpn should
  _NOT_ have the vpn.fedoraproject.org search until after it has been added
  o the vpn (if it ever does)

Phoenix
  ::
 
    search phx2.fedoraproject.org vpn.fedoraproject.org fedoraproject.org

Phoenix in the QA network: 
  ::

    search qa.fedoraproject.org vpn.fedoraproject.org phx2.fedoraproject.org fedoraproject.org

Non-Phoenix
  ::
 
    search vpn.fedoraproject.org fedoraproject.org

The idea here is that we can, when need be, setup local domains to contact
instead of having to go over the VPN directly but still have sane configs.
For example if we tell the proxy server to hit "app1" and that box is in
PHX, it will go directly to app1, if its not, it will go over the vpn to
app1.

