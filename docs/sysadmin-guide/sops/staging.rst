.. title: Infrastructure Ansible Staging SOP
.. slug: infra-staging-ansible
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

===================
Ansible Staging SOP
===================

Owner
	Fedora Infrastructure Team =

Contact
	#fedora-admin, sysadmin-main =

Location
	Mostly in PHX2 =

Servers
	*stg* =

Purpose
	Staging environment to test changes to apps and create initial Ansible configs.  =

Introduction
============

Fedora uses a set of staging servers for several purposes: 

* When applications are initially being deployed, the staging version of 
  those applications are setup with a staging server that is used to create the 
  initial Ansible configuration for the application/service. 

* Established applications/services use staging for testing. This testing includes: 

  - Bugfix updates 
  - Configuration changes managed by Ansible 
  - Upstream updates to dependent packages (httpd changes for example)

Goals
=====

The staging servers should be self contained and have all the needed databases and such 
to function. At no time should staging resources talk to production instances. We use firewall
rules on our production servers to make sure no access is made from staging. 

Staging instances do often use dumps of production databases and data, and
thus access to resources in staging should be controlled as it is in
production. 

DNS and naming
==============

All staging servers should be in the ``stg.phx2.fedoraproject.org`` domain. 
/etc/hosts files are used on stg servers to override dns in cases where staging resources 
should talk to the staging version of a service instead of the production one. 
In some cases, one staging server may be aliased to several services or applications that 
are on different machines in production. 

Syncing databases
=================

Syncing FAS
-----------
Sometimes you want to resync the staging fas server with what's on
production. To do that, dump what's in the production db and then import
it into the staging db. Note that resyncing the information will remove
any of the information that has been added to the staging fas servers. So
it's good to mention that you're doing this on the infra list or to people
who you know are working on the staging fas servers so they can either
save their changes or ask you to hold off for a while.

On db01::

  $ ssh db01
  $ sudo -u postgres pg_dump -C fas2 |xz -c fas2.dump.xz
  $ scp fas2.dump.xz db02.stg:

On fas01.stg (postgres won't drop the database if something is accessing it)
(ATM, fas in staging is not load balanced so we only have to do this on one server)::

  $ sudo /etc/init.d/httpd stop

On db02.stg::

  $ echo 'drop database fas2' |sudo -u postgres psql
  $ xzcat fas2.dump.xz | sudo -u postgres psql

On fas01.stg::

  $ sudo /etc/init.d/httpd start

Other databases behave similarly. 

External access
===============

There is http/https access from the internet to staging instances to allow testing.
Simply replace the production resource domain with stg.fedoraproject.org and
it should go to the staging version (if any) of that resource. 

Ansible and Staging
=================== 

All staging machine configurations is now in the same branch 
as master/production. 

There is a 'staging' environment - Ansible variable "env" is equal to
"staging" in playbooks for staging things. This variable can be used
to differentiate between producion and staging systems.

Workflow for staging changes
============================

1. If you don't need to make any Ansible related config changes, don't
    do anything. (ie, a new version of an app that uses the same config
    files, etc). Just update on the host and test. 

2. If you need to make Ansible changes, either in the playbook of the
    application or outside of your module:

    - Make use of files ending with .staging (see resolv.conf in global for
      an example). So, if there's persistent changes in staging from
      production like a different config file, use this. 

    - Conditionalize on environment: 

    - name: your task::
      ...
      when: env == "staging"

    - name: production-only task::

       ...
       when: env != "staging"

    - These changes can stay if they are helpful for further testing down
      the road. Ideally normal case is that staging and production are
      configure in the same host group from the same Ansible playbook.

Time limits on staging changes
===============================

There is no hard limit on time spent in staging, but where possible we should 
limit the time in staging so we are not carrying changes from production for a
long time and possible affecting other staging work.
