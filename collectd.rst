.. title: Collectd SOP
.. slug: collectd
.. date: 2016-03-22
.. taxonomy: Contributors/Infrastructure

============
Collectd SOP
============

Collectd ( https://collectd.org/ ) is a client/server setup that gathers system information 
from clients and allows the server to display that information over various time periods.

Our server instance runs on log01.phx2.fedoraproject.org and most other servers run 
clients that connect to the server and provide it with data.

========

1. Contact Information
2. Collectd info

Contact Information
===================

Owner
         Fedora Infrastructure Team
Contact
         #fedora-admin
Location
        https://admin.fedoraproject.org/collectd/
Servers
    log01 and all/most other servers as clients
Purpose
    provide load and system information on servers.

Configuration
=============

The collectd roles configure collectd on the various machines:

collectd/base - This is the base client role for most servers. 
collectd/server - This is the server for use on log01. 
collectd/other - There's various other subroles for different types of clients. 

Web interface
=============

The server web interface is available at:

https://admin.fedoraproject.org/collectd/

Restarting
==========

collectd runs as a normal systemd or sysvinit service, so you can: 
systemctl restart collectd or service collectd restart
to restart it. 

Removing old hosts
==================

Collectd keeps information around until it's deleted, so you may need to 
sometime go remove data from a host or hosts thats no longer used. 
To do this: 

1. Login to log01
2. cd /var/lib/collectd/rrd
3. sudo rm -rf oldhostname

Bug reporting
=============

Collectd is in Fedora/EPEL and we use their packages, so report bugs to bugzilla.redhat.com.
