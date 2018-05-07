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
    6.2. Generate an OpenID Connect token

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Primary upstream contact
    Patrick Uiterwijk - FAS: puiterwijk
Backup upstream contact
    Simo Sorce - FAS: simo (irc: simo)
    Howard Johnson - FAS: merlinthp (irc: MerlinTHP)
    Rob Crittenden - FAS: rcritten (irc: rcrit)
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


Generate an OpenID Connect token
--------------------------------

There is a handy script in the Ansible project under ``scripts/generate-oidc-token`` that can help
you generate an OIDC token. It has a self-explanatory ``--help`` argument, and it will print out
some SQL that you can run against Ipsilon's database, as well as the token that you seek.

The ``SERVICE_NAME`` (the required positional argument) is the name of the application that wants to
use the token to perform actions against another service.

To generate the scopes, you can visit our authentication_ docs and find the service you want the
token to be used for. Each service has a base namespace (a URL) and one or more scopes for that
namespace. To form a scope for this script, you concatenate the namespace of the service with the
scope you want to grant the service. You can provide the script the -s flag multiple times if you
want to grant more than one scope to the same token.

As an example, to give Bodhi access to create waivers in WaiverDB, you can see that the base
namespace is ``https://waiverdb.fedoraproject.org/oidc/`` and that there is a ``create-waiver``
scope. You can run this to generate Ipsilon SQL and a token with that scope::

    [bowlofeggs@batcave01 ansible][PROD]$ ./scripts/generate-oidc-token bodhi -e 365 -s https://waiverdb.fedoraproject.org/oidc/create-waiver

    Run this SQL against Ipsilon's database:

    --------START CUTTING HERE--------
    BEGIN;
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','username','bodhi@service');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','security_check','-ptBqVLId-kUJquqkVyhvR0DbDULIiKp1eqbXqG_dfVK9qACU6WwRBN3-7TRfoOn');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','client_id','bodhi');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','expires_at','1557259744');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','type','Bearer');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','issued_at','1525723744');
    insert into token values ('2a5f2dff-4e93-4a8d-8482-e62f40dce046','scope','["openid", "https://someapp.fedoraproject.org/"]');
    COMMIT;
    -------- END CUTTING HERE --------


    Token: 2a5f2dff-4e93-4a8d-8482-e62f40dce046_-ptBqVLId-kUJquqkVyhvR0DbDULIiKp1eqbXqG_dfVK9qACU6WwRBN3-7TRfoOn

Once you have the SQL, you can run it against Ipsilon's database, and you can provide the token to
the application through some secure means (such as putting into Ansible's secrets and telling the
requestor the Ansible variable they can use to access it.)

.. authentication: https://fedoraproject.org/wiki/Infrastructure/Authentication
