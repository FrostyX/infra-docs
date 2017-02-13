.. title: Gitweb Infrastructure SOP
.. slug: infra-gitweb
.. date: 2011-08-23
.. taxonomy: Contributors/Infrastructure

=========================
Gitweb Infrastructure SOP
=========================

Gitweb-caching is the web interface we use to expose git to the web at
http://git.fedorahosted.org/git/

Contact Information
===================

Owner
  Fedora Infrastructure Team
Contact
  #fedora-admin, sysadmin-hosted
Location
  Serverbeach
Servers
  hosted[1-2]
Purpose
  Http access to git sources.

Basic Function
==============

- Users go to [46]http://git.fedorahosted.org/git/ 

- Pages are generated from cache stored in ``/var/cache/gitweb-caching/``. 
  
- The website is exposed via ``/etc/httpd/conf.d/git.fedoraproject.org.conf``. 
  
- Main config file is ``/var/www/gitweb-caching/gitweb_config.pl``. 
  This pulls git repos from /git/.

