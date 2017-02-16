.. title: Ipsilon Infrastucture SOP
.. slug: infra-ipsilon
.. date: 2016-03-21
.. taxonomy: Contributors/Infrastructure

==========================
Ipsilon Infrastructure SOP
==========================



Contents
========

1. Contact Information
2. Description
3. Known Issues
4. ReStarting
5. Configuration
6. Common actions
    6.1. Registering OpenID Connect Scopes

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	 Phoenix
Servers
	ipsilon01.phx2.fedoraproject.org ipsilon02.phx2.fedoraproject.org ipsilion01.stg.phx2.fedoraproject.org. 
	 
Purpose
	Ipsilon is our central authentication service that is used to authenticate users agains FAS. It is seperate from FAS.	

Description
===========

Ipsilon is our central authentication agent that is used to authenticate users agains FAS. It is seperate from FAS. The only service that is not using this currently is the wiki. It is a web service that is presented via httpd and is load balanced by our standard haproxy setup.

Known issues
==============

No known issues at this time. There is not currently a logout option for ipsilon, but it is not considered an issue. If group memberships are updated in ipsilon the user will need to wait a few minutes for them to replicate to the all the systems.

Restarting
===============

To restart the application you simply need to ssh to the servers for the problematic region and issue an 'service httpd restart'. This should rarely be required.

Configuration
================

Configuration is handled by the ipsilon.yaml playbook in Ansible. This can also be used to reconfigure application, if that becomes nessecary.

Common actions
==============
This section describes some common configuration actions.

OpenID Connect Scope Registration
---------------------------------
As documented on https://fedoraproject.org/wiki/Infrastructure/Authentication, application developers can request their own scopes.
When a request for this comes in, look in ansible/roles/ipsilon/files/oidc_scopes/ and copy an example module.
Copy this to a new file, so we have a file per scope set.
Fill in the information:

  - name is an Ipsilon-internal name. This should not include any spaces
  - display_name is the name that is displayed to the category of scopes to the user
  - scopes is a dictionary with the full scope identifier (with namespace) as keys.
    The values are dicts with the following keys:

        - display_name: The complete display name for this scope. This is what the user gets shown to accept/reject
        - claims: A list of additional "claims" (pieces of user information) an application will get when the user
        - consents to this scope. For most scopes, this will be the empty list.

In ansible/roles/ipsilon/tasks/main.yml, add the name of the new file (without .py) to the with_items of
"Copy OpenID Connect scope registrations").
To enable, open ansible/roles/ipsilon/templates/configuration.conf, and look for the lines starting with
"openidc enabled extensions".
Add the name of the plugin (in the "name" field of the file) to the environment this scopeset has been requested for.
Run the ansible ipsilon.yml playbook.
