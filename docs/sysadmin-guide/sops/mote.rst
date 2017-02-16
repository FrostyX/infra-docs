.. title: mote SOP
.. slug: infra-mote
.. date: 2015-06-13
.. taxonomy: Contributors/Infrastructure

===========
mote SOP
===========

mote is a MeetBot log wrangler, providing
an user-friendly interface for viewing logs produced
by Fedora's IRC meetings.

Production instance: http://meetbot.fedoraproject.org/
Staging instance:    http://meetbot.stg.fedoraproject.org

Contents
========
1.  Contact information
2.  Deployment
3.  Description
4.  Configuration
5.  Database
6.  Managing mote
7.  Suspespending mote operation
8.  Changing mote's name and category definitions

Contact Information
===================
Owner
        cydrobolt
Contact
        #fedora-admin
Location
        Fedora Infrastructure
Purpose
        IRC meeting coordination


Deployment
==========
If you have access to rbac-playbook::

      sudo rbac-playbook groups/value.yml

Forcing Reload
==============

There is a playbook that can force mote to update its cache
in case it gets stuck somehow::

      sudo rbac-playbook manual/rebuild/mote.yml

Doing Upgrades
==============

Put a new copy of the mote rpm in the infra repo and run::

      sudo rbac-playbook manual/upgrade/mote.yml

Description
===========
mote is a Python webapp running on Flask with mod_wsgi.
It can be used to view past logs, browse meeting minutes, or
glean other information relevant to Fedora's IRC meetings.
It employs a JSON file store cache, in addition to a 
memcached store which is currently not in use with
Fedora infrastructure.


Configuration
=============
mote configuration is located in ``/etc/mote/config.py``. The
configuration contains all configurable items for all mote services.
Alterations to configuration that aren't temporary should be done through ansible playbooks.
Configuration changes have no effect on running services -- they 
need to be restarted, which can be done using the playbook.


Database
========
mote does not currently utilise any databases, although it uses a 
file store in Fedora Infrastructure and has an optional memcached store
which is currently unused.

Managing mote
=============
mote is ran using mod_wsgi and httpd, hence, you must
manage the ``httpd`` service to change mote's status.

Suspespending mote operation
============================
mote can be stopped by stopping the ``httpd`` service::

    service httpd stop

Changing mote's name and category definitions
=============================================
mote uses a set of JSON name and category definitions to provide
friendly names, aliases, and listings on its interface.
These definitions can be located in mote's GitHub repository,
and need to be pulled into ansible in order to be deployed.

These files are ``name_mappings.json`` and ``category_mappings.json``.
To deploy an update to these definitions, place the updated name and
category mapping files in ``ansible/roles/mote/templates``. Run
the playbook in order to deploy your changes.
