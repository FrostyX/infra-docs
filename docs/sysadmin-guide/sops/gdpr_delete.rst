.. title: GDPR Delete
.. slug: infra-docs
.. date: 2018-06-19
.. taxonomy: Contributors/Infrastructure

===============
GDPR Delete SOP
===============

This SOP covers how Fedora Infrastructure handles General Data Protection Regulation (GDPR) Delete
Requests. It contains information about how system administrators will use tooling to
respond to Delete requests, as well as how application developers can integrate their applications with that
tooling.


Contact Information
===================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin
Persons
 nirik
Location
 Phoenix
Servers
 batcave01.phx2.fedoraproject.org
 Various application servers, which will run scripts to delete data.
Purpose
 Respond to Delete requests.


Responding to a SAR
===================

This section covers how a system administrator will use our ``gdpr-delete.yml`` playbook to respond to a
Delete request.

When processing a Delete request, perform the following steps:

0. Verify that the requester is who they say they are. If the request came in email ask them 
   to file an issue at https://pagure.io/fedora-pdr/new_issue 
   Use the following in email reply to them: 

   ``In order to verify your identity, please file a new issue at
     https://pagure.io/fedora-pdr/new_issue using the appropriate issue type.

     Please note this form requires you to sign in to your account to verify
     your identity.``

   If they do not have a FAS account, indicate to them that there is no data to be deleted.
   Use this response:

   ``Your request for deletion has been reviewed. Since there is no related
   account in the Fedora Account System, the Fedora infrastructure does
   not store data relevant for this deletion request. Note that some
   public content related to Fedora you may have previously submitted
   without an account, such as to public mailing lists, is not deleted
   since accurate maintenance of this data serves both Fedora's and the
   public interest as well as the open source community.``
   
1. Identify the users FAS account name. The Delete playbook will use this FAS account to delete
   the required data. Update the fedora-pdr issue saying the request has been received.
   There is a 'quick response' in the pagure issue tracker to note this. 

2. Login to FAS and clear the ``Telephone number`` entry, set Country to ``Other``, clear
   ``Lattitude`` and ``Longitude`` and ``IRC Nick`` and ``GPG Key ID`` and set ``Time Zone`` to UTC
   and ``Locale`` to ``en`` and set the user status to ``disabled``. If the user is not in cla_done 
   plus one group, you are done. Update the ticket and close it. This step will be folded into the 
   following one once we implement it.

3. If the user is in cla_done + one group, they may have additional data:
   Run the gdpr delete playbook on ``batcave01``. You will need to define one Ansible variable for the
   playbook. ``sar_fas_user`` will be the FAS username of the user.

     $ sudo ansible-playbook playbooks/manual/gdpr/delete.yml -e gdpr_delete_fas_user=bowlofeggs

   After the script completes, update the ticket that the request is completed and close it.
   There is a 'quick response' in the pagure issue tracker to note this. 

Integrating an application with our delete playbook
===================================================

This section covers how an infrastructure application can be configured to integrate with our
``delete.yml`` playbook. To integrate, you must create a script and Ansible variables so that your
application is compatible with this playbook.


Script
------

You need to create a script and have your project's Ansible role install that script somewhere
(most likely on a host from your project - for example fedocal's is going on ``fedocal01``.)
It's not a bad idea to put your script into your upstream project. This script should accept 
one environment variable as input: ``GDPR_DELETE_USERNAME``.  This will be a FAS username.

Some scripts may need secrets embedded in them - if you must do this be careful to install the
script with ``0700`` permissions, ensuring that only ``gdpr_delete_script_user`` (defined below) can run
them. Bodhi worked around this concern by having the script run as ``apache`` so it could read
Bodhi's server config file to get the secrets, so it does not have secrets in its script.


Variables
---------

In addition to writing a script, you need to define some Ansible variables for the host that
will run your script:

=================== ===================================================== ======================
Variable            Description                                           Example
------------------- ----------------------------------------------------- ----------------------
``gdpr_delete_script``      The full path to the script.                          ``/usr/bin/fedocal-delete``
``gdpr_delete_script_user`` The user the script should be run as                  ``apache``
=================== ===================================================== ======================

You also need to add the host that the script should run on to the ``[gdpr_delete]`` group in
``inventory/inventory``::

    [gdpr_delete]
    fedocal01.phx2.fedoraproject.org
