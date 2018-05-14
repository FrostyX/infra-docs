.. title: GDPR SAR
.. slug: infra-bodhi
.. date: 2018-05-14
.. taxonomy: Contributors/Infrastructure

============
GDPR SAR SOP
============

This SOP covers how Fedora Infrastructure handles General Data Protection Regulation (GDPR) Subject
Access Requests (SAR). It contains information about how system administrations will use tooling to
respond to SARs, as well as how application developers can integrate their applications with that
tooling.


Contact Information
===================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin
Persons
 bowlofeggs
Location
 Phoenix
Servers
 batcave01.phx2.fedoraproject.org
 Various application servers, which will run scripts to collect SAR data.
Purpose
 Respond to SARs.


Responding to a SAR
===================

This section covers how a system administrator will use our ``sar.yml`` playbook to respond to a
SAR.

When processing a SAR, perform the following steps:

0. Verify that the requester is who they say they are.
1. Identify an e-mail address for the requester, and if applicable, their FAS account name. The SAR
   playbook will use both of these since some applications have data associated with FAS accounts
   and others have data associated with e-mail addresses.
2. Run the SAR playbook on ``batcave01``. You will need to define three Ansible variables for the
   playbook. ``sar_fas_user`` will be the FAS username, if applicable; this may be omitted if the
   requester does not have a FAS account. ``sar_email`` will be the e-mail address associated with
   the user. ``sar_tar_output_path`` will be the path you want the playbook to write the resulting
   tarball to, and should have a ``.tar.gz`` extension. For example, if ``bowlofeggs`` submitted a
   SAR and his e-mail address is ``bowlof@eggs.biz``, you might run the playbook like this::

     $ sudo ansible-playbook playbooks/manual/gdpr/sar.yml -e sar_fas_user=bowlofeggs \
         -e sar_email=bowlof@eggs.biz -e sar_tar_output_path=/home/bowlofeggs/bowlofeggs.tar.gz

3. Transmit the tarball to the requester.


Integrating an application with our SAR playbook
================================================

This section covers how an infrastructure application can be configured to integrate with our
``sar.yml`` playbook. To integrate, you must create a script and Ansible variables so that your
application is compatible with this playbook.


Script
------

You need to create a script and have your project's Ansible role install that script somewhere
(most likely on a host from your project - for example Bodhi's is going on ``bodhi-backend02``.)
It's not a bad idea to put your script into your upstream project - there are plans for upstream
Bodhi to ship ``bodhi-sar``, for example. This script should accept two environment variables as
input: ``SAR_USERNAME`` and ``SAR_EMAIL``. Not all applications will use both, so do what makes
sense for your application. The first will be a FAS username and the second will be an e-mail
address. Your script should gather the required information related to those identifiers and print
it in a machine readable format to stdout. Bodhi, for example, prints information to stdout in
``JSON``.

Some scripts may need secrets embedded in them - if you must do this be careful to install the
script with ``0700`` permissions, ensuring that only ``sar_script_user`` (defined below) can run
them. Bodhi worked around this concern by having the script run as ``apache`` so it could read
Bodhi's server config file to get the secrets, so it does not have secrets in its script.


Variables
---------

In addition to writing a script, you need to define some Ansible variables for the host that
will run your script:

=================== ===================================================== ======================
Variable            Description                                           Example
------------------- ----------------------------------------------------- ----------------------
``sar_script``      The full path to the script.                          ``/usr/bin/bodhi-sar``
``sar_script_user`` The user the script should be run as                  ``apache``
``sar_output_file`` The name of the file to write into the output tarball ``bodhi.json``
=================== ===================================================== ======================

You also need to add the host that the script should run on to the ``[sar]`` group in
``inventory/inventory``::

    [sar]
    bodhi-backend02.phx2.fedoraproject.org
