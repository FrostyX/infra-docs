.. title: SSH Access SOP
.. slug: infra-ssh-access
.. date: 2012-09-24
.. taxonomy: Contributors/Infrastructure

.. _ssh-sop:

=============================
SSH Access Infrastructure SOP
=============================

Contents
========

1. Contact Information
2. Introduction
3. SSH configuration
4. SSH Agent forwarding
5. Troubleshooting

Contact Information
===================

Owner
	 sysadmin-main
Contact
	 #fedora-admin or admin@fedoraproject.org
Location
	 PHX2
Servers
	 All PHX2 and VPN Fedora machines
Purpose
	 Access via ssh to Fedora project machines.

Introduction
============

This page will contain some useful instructions about how you can safely
login into Fedora PHX2 machines successfully using a public key
authentication. As of 2011-05-27, all machines require a SSH key to
access. Password authentication will no longer work. Note that this SOP
has nothing to do with actually gaining access to specific machines. For
that you MUST be in the correct group for shell access to that machine.
This SOP simply describes the process once you do have valid and
appropriate shell access to a machine.

SSH configuration
=================
First of all: (on your local machine)::

  vi ~/.ssh/config

.. note::
  This file, and any keys, need to be chmod 600, or you will get a "Bad owner or
  permissions" error. The .ssh directory must be mode 700.

then, add the following::

  Host bastion.fedoraproject.org
    User FAS_USERNAME
    ProxyCommand none
    ForwardAgent no
  Host *.phx2.fedoraproject.org *.qa.fedoraproject.org 10.5.125.* 10.5.126.* 10.5.127.* *.vpn.fedoraproject.org
    User FAS_USERNAME
    ProxyCommand ssh -W %h:%p bastion.fedoraproject.org

One slight annoyance with this method is that you must include the
.phx2.fedoraproject.org part when you SSH to Fedora machines in order for
the connection to be tunneled through bastion.

To avoid this You can add aliases for each of the Fedora machines you login to by
modifying the second Host line::

  Host *.phx2.fedoraproject.org 10.5.125.* 10.5.126.* 10.5.127.* *.vpn.fedoraproject.org batcave01 noc01 # list all hosts here

How ProxyCommand works?

A connection is established to the bastion host

+-------+            +--------------+
|  you  | ---ssh---> | bastion host |
+-------+            +--------------+
Bastion host establish a connction to the target server

+--------------+          +--------+
| bastion host | -------> | server |
+--------------+          +--------+
Your client then connects through the Bastion and reaches the target server

+-----+                  +--------------+                +--------+
| you |                  | bastion host |                | server |
|     | ===ssh=over=bastion============================> |        |
+-----+                  +--------------+                +--------+

PuTTY SSH configuration
=======================

You can configure Putty the same way by doing this:

0.In the session section type batcave01.phx2.fedoraproject.org port 22
1.In Connection:Data enter your FAS_USERNAME
2.In Connection:Proxy add the proxy settings
.ProxyHostname is bastion.fedoraproject.org
.Port 22
.Username FAS_USERNAME
.Proxy Command plink %user@%proxyhost %host:%port
3.In Connection:SSH:Auth remember to insert the same key file for authentication you have used on FAS profile 

SSH Agent forwarding
====================

You should normally have::

  ForwardAgent no

For Fedora hosts (this is the default in OpenSSH). You can override this
on a per-session basis by using '-A' with ssh. SSH agents could be misused
if you connect to a compromised host with forwarding on (the attacker can
use your agent to authenticate them to anything you have access to as long
as you are logged in). Additionally, if you do need SSH agent forwarding
(say for copying files between machines), you should remember to logout as
soon as you are done to not leave your agent exposed.

Troubleshooting
===============

* 'channel 0: open failed: administratively prohibited: open failed'
    
    If you receive this message for a machine proxied through bastion, then
    bastion was unable to connect to the host. This most likely means that
    tried to SSH to a nonexistent machine. You can debug this by trying to
    connect to that machine from bastion.
 
* if your local username is different from the one registered in FAS,
    please remember to set up a User variable (like above) where you
    specify your FAS username. If that's missing SSH will try to login by
    using your local username, thus it will fail.
    
* ssh -vv is very handy for debugging what sections are matching and
    what are not.
 
* If you get access denied several times in a row, please consult with
    #fedora-admin. If you try too many times with an invalid config your
    IP could be added to denyhosts.
 
* If you are running an OpenSSH version less than 5.4, then the -W
    option is not available. In that case, use the following ProxyCommand
    line instead::

      ProxyCommand ssh -q bastion.fedoraproject.org exec nc %h %p
