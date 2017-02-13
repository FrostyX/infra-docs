.. title: geoip-city-wsgi SOP
.. slug: geoip-city-wsgi
.. date: 2017-01-30
.. taxonomy: Contributors/Infrastructure


====================
geoip-city-wsgi SOP
====================

A simple web service that return geoip information as JSON-formatted dictionary in utf-8. Particularly, it's used by anaconda[1] to get the most probable territory code, based on the public IP of the caller.

Contents
========

1. Contact Information
2. Basic Function
3. Ansible Roles
4. Apps depending of geoip-city-wsgi
5. Documentation Links


Contact Information
====================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-admin, #fedora-noc
Location
	https://geoip.fedoraproject.org
Servers
	sundries*, sundries*-stg
Purpose
	A simple web service that return geoip information as JSON-formatted dictionary in utf-8. Particularly, it's used by anaconda[1] to get the most probable territory code, based on the public IP of the caller.
	
Basic Function
==============

- Users go to https://geoip.fedoraproject.org/city 
  
- The website is exposed via ``/etc/httpd/conf.d/geoip-city-wsgi-proxy.conf``. 

- Return a string with geoip information with syntax as JSON-formatted dict in utf8

- It also currently accepts one override: ?ip=xxx.xxx.xxx.xxx, e.g. https://geoip.fedoraproject.org/city?ip=18.0.0.1 which then uses the passed IP address instead of the determined IP address of the client. 


Ansible Roles
==============
The geoip-city-wsgi role https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/roles/geoip-city-wsgi
is present in sundries playbook https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/playbooks/groups/sundries.yml

the proxy task are present in
https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/playbooks/include/proxies-reverseproxy.yml

Apps depending of geoip-city-wsgi
=================================
unknown

Documentation Links
===================

app:    https://geoip.fedoraproject.org
source: https://github.com/fedora-infra/geoip-city-wsgi
bugs:   https://github.com/fedora-infra/geoip-city-wsgi/issues
Role:	https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/roles/geoip-city-wsgi
[1]     https://fedoraproject.org/wiki/Anaconda

