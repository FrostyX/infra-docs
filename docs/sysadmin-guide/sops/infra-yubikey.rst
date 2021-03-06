.. title: Infrastructure Yubikey SOP
.. slug: infra-yubikey
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

==========================
Infrastructure/SOP/Yubikey
==========================

This document describes how yubikey authentication works

Contents
========

1. Contact Information
2. User Information
3. Host Admins

	1. pam_yubico

4. Server Admins

	1. Basic architecture
	2. ykval
	3. ykksm
	4. Physical Yubikey info

5. fas integration

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-main

Location
	Phoenix

Servers
	fas*, db02

Purpose
	Provides yubikey authentication in Fedora

Config Files
============
* ``/etc/httpd/conf.d/yk-ksm.conf``
* ``/etc/httpd/conf.d/yk-val.conf``
* ``/etc/ykval/ykval-config.php``
* ``/etc/ykksm/ykksm-config.php``
* ``/etc/fas.cfg``

User Information
================

See [57]Infrastruture/Yubikey

Host Admins
===========

pam_yubico

Generated from fas, the /etc/yubikeyid works like a authroized_keys file
and maps valid keys to users. It is downloaded from FAS:

[58]https://admin.fedoraproject.org/accounts/yubikey/dump

Server Admins
=============
Basic architecture
------------------
Yubikey authentication takes place in 3 basic phases.

1. User presses yubikey which generates a one time password
2. The one time password makes its way to the yk-val application which
    verifies it is not a replay
3. yk-val passes that otp on to the yk-ksm application which verifies the
    key itself is a valid key

If all of those steps succeed, the ykval application sends back an OK and
authentication is considered successful. The two applications are defined
below, if either of them is unavailable, yubikey authentication will fail.

ykval
``````

Database: db02:ykval

The database contains 3 tables. clients: just a valid client. These are
not users, these are systems able to authenticate against ykval. In our
case Fedora is the only client so there's just one entry here queue: Used
for distributed setups (we don't do this) yubikeys: maps which yubikey
belongs to which user

ykval is installed on fas* and is located at:
[59]http://localhost/yk-val/verify

Purpose: Is to map keys to users and protect against replay attacks

ykksm
``````
Database: db02:ykksm

The database contains one table: yubikeys: maps who created keys, what key
was created, when, and the public name and serial number, whether its
active, etc.

ykksm is installed on fas* at [60]http://localhost/yk-ksm

Purpose: verify if a key is a valid known key or not. Nothing contacts
this service directly except for ykval. This should be considered the
“high security” portion of the system as access to this table would allow
users to make their own yubikeys.

Physical Yubikey info
``````````````````````

The actual yubikey contains information to generate a one time password.
The important bits to know are the begining of the otp contains the
identifier of the key (used similar to how ssh uses authorized_keys) and
note the rest of it contains lots of bits of information, including a
serial incremental.

Sample key: ``ccccfcdaivjrvdhvzfljbbievftnvncljhibkulrftt``

Breaking this up, the first 12 characters are the identifier. This can be
considered 'public'

ccccfcdaivj rvdhvzfljbbievftnvncljhibkulrftt

The second half is the otp part.

fas integration
===============
Fas integration has two main parts. First is key generation, the next is
activation. The fas-plugin-yubikey contains the bits for both, and
verification. Users call on this page to generate the key info:

[61]https://admin.fedoraproject.org/accounts/yubikey/genkey

The fas password field automatically detects whether someone is using a
otp or a regular password. It then sends otp requests to yk-val for
verification.

