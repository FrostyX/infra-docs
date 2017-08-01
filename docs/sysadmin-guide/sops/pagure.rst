.. title: Pagure Infrastucture SOP
.. slug: infra-pagure
.. date: 2017-07-05
.. taxonomy: Contributors/Infrastructure

=========================
Pagure Infrastructure SOP
=========================

Pagure is a code hosting and management site.

Contents
========

1. Contact Information
2. Description
3. When unresponsive
4. Git repo locations
5. Services and what they do

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-apps
Location
	 OSUOSL
Servers
	 pagure01, pagure-stg01
Purpose
	 Source code and issue tracking

Description
===========

Pagure ( https://pagure.io/pagure ) is a source code management and issue 
tracking application. Its written in flask. It uses: celery, redis, postgresql, 
and pygit2. 


When unresponsive
=================

Sometimes pagure will stop responding, even though it's still running. 
You can issue a 'systemctl reload httpd' and that will usually get it 
running again. 

Git repo locations
==================

* Main repos are in /srv/git/repositories/<projectname>
* issue/ticket repos are under /srv/git/repositories/tickets/<projectname>
* Docs are under /srv/git/repositories/docs/<projectname>
* Releases (not a git repo) are under /var/www/releases/

Services and what they do
=========================

* pagure service is the main flask application, it runs from httpd wsgi.
* pagure_ci service talks to jenkins or other CI for testing PR's
* pagure_ev service talks to websockets and updates issues and comments live for users. 
* pagure_loadjson service takes issues loads from pagure-importer and processes them. 
* pagure_logcom service handles logging. 
* pagure_milter processes email actions. 
* pagure_webhook service processes webhooks to notify about changes. 
* pagure worker service updates git repos with changes. 
