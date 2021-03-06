.. title: haproxy Infrastructure SOP
.. slug: infra-haproxy
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

==========================
Haproxy Infrastructure SOP
==========================

haproxy is an application that does load balancing at the tcp layer or at
the http layer. It can do generic tcp balancing but it does specialize in
http balancing. Our proxy servers are still running apache and that is
what our users connect to. But instead of using mod_proxy_balancer and
ProxyPass balancer://, we do a ProxyPass to [45]http://localhost:10001/ or
[46]http://localhost:10002/. haproxy must be told to listen to an
individual port for each farm. All haproxy farms are listed in
/etc/haproxy/haproxy.cfg.

Contents
========

1. Contact Information
2. How it works
3. Configuration example
4. Stats
5. Advanced Usage

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin, sysadmin-main, sysadmin-web group
Location:
  Phoenix, Tummy, Telia
Servers: 
  proxy1, proxy2, proxy3, proxy4, proxy5
Purpose: 
  Provides load balancing from the proxy layer to our application
  layer.

How it works
============

haproxy is a load balancer. If you're familiar, this section won't be that
interesting. haproxy in its normal usage acts just like a web server. It
listens on a port for requests. Unlike most webservers though it then
sends that request to one of our back end application servers and sends
the response back. This is referred to as reverse proxying. We typically
configure haproxy to send check to a specific url and look for the
response code. If this url isn't sent, it just does basic checks to /. In
most of our configurations we're using round robin balancing. IE, request
1 goes to app1, request2 goes to app2, request 3 goes to app3 request 4
goes to app1, and the whole process repeats.

.. warning:: 
  These checks do add load to the app servers. As well as additional
  connections. Be smart about which url you're checking as it gets checked
  often. Also be sure to verify the application servers can handle your new
  settings, monitor them closely for the hour or two after you make changes.

Configuration example
=====================

The below example is how our fedoraproject wiki could be configured. Each
application should have its own farm. Even though it may have an identical
configuration to another farm, this allows easy addition and subtraction
of specific nodes when we need them.::

  listen  fpo-wiki 0.0.0.0:10001
  balance roundrobin
  server  app1 app1.fedora.phx.redhat.com:80 check inter 2s rise 2 fall 5
  server  app2 app2.fedora.phx.redhat.com:80 check inter 2s rise 2 fall 5
  server  app4 app4.fedora.phx.redhat.com:80 backup check inter 2s rise 2 fall 5
  option  httpchk GET /wiki/Infrastructure

* The first line "listen ...." Says to create a farm called 'fpo-wiki'.
  Listening on all IP's on port 10001. fpo-wiki can be arbitrary but make it
  something obvious. Aside from that the important bit is :10001. Always
  make sure that when creating a new farm, its listening on a unique port.
  In Fedora's case we're starting at 10001, and moving up by one. Just check
  the config file for the lowest open port above 10001.

* The next line "balance roundrobin" says to use round robin balancing.

* The server lines each add a new node to the balancer farm. In this case
  the wiki is being served from app1, app2 and app4. If the wiki is
  available at [53]http://app1.fedora.phx.redhat.com/wiki/ Then this config
  would be used in conjunction with "RewriteRule ^/wiki/(.*)
  [54]http://localhost:10001/wiki/$1 [P,L]".

* 'server' means we're adding a new node to the farm

* 'app1' is the worker name, it is analagous to fpo-wiki but should
   match shorthostname of the node to make it easy to follow.

* 'app1.fedora.phx.redhat.com:80' is the hostname and port to be
  contacted.

* 'check' means to check via bottom line "option httpchk GET
  /wiki/Infrastructure" which will use /wiki/Infrastructure to verify
  the wiki is working. If that URL fails, that entire node will be taken
  out of the farm mix.

* 'inter 2s' means to check every 2 seconds. 2s is the same as 2000 in
  this case.

* 'rise 2' means to not put this node back in the mix until it has had
  two successful connections in a row. haproxy will continue to check
  every 2 seconds whether a node is up or down

* 'fall 5' means to take a node out of the farm after 5 failures.

* 'backup' You'll notice that app4 has a 'backup' option. We don't
  actually use this for the wiki but do for other farms. It basically
  means to continue checking and treat this node like any other node but
  don't send it any production traffic unless the other two nodes are
  down.

All of these options can be tweaked so keep that in mind when changing or
building a new farm. There are other configuration options in this file
that are global. Please see the haproxy documentation for more info::

  /usr/share/doc/haproxy-1.3.14.6/haproxy-en.txt

Stats
=====

In order to view the stats for a farm please see the stats page. Each
proxy server has its own stats page since each one is running its own
haproxy server. To view the stats point your browser to
https://admin.fedoraproject.org/haproxy/shorthostname/ so proxy1 is at
https://admin.fedoraproject.org/haproxy/proxy1/ The trailing / is
important.

* https://admin.fedoraproject.org/haproxy/proxy1/
* https://admin.fedoraproject.org/haproxy/proxy2/
* https://admin.fedoraproject.org/haproxy/proxy3/
* https://admin.fedoraproject.org/haproxy/proxy4/
* https://admin.fedoraproject.org/haproxy/proxy5/

Advanced Usage
==============

haproxy has some more advanced usage that we've not needed to worry about
yet but is worth mentioning. For example, one could send users to just one
app server based on session id. If user A happened to hit app1 first and
user B happened to hit app4 first. All subsequent requests for user A
would go to app1 and user B would go to app4. This is handy for
applications that cannot normally be balanced because of shared storage
needs or other locking issues. This won't solve all problems though and
can have negative affects for example when app1 goes down user A would
either lose their session, or be unable to work until app1 comes back up.
Please do some great testing before looking in to this option.

