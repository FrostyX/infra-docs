.. title: Two Factor Auth
.. slug: fas-two-factor
.. date: 2013-09-19 updated: 2016-03-11
.. taxonomy: Contributors/Infrastructure

===============
Two factor auth
===============

Fedora Infrastructure has implemented a form of two factor auth for people who
have sudo access on Fedora machines.  In the future we may expand this to
include more than sudo but this was deemed to be a high value, low hanging 
fruit.

----------------
Using two factor
----------------

http://fedoraproject.org/wiki/Infrastructure_Two_Factor_Auth

To enroll a Yubikey, use the fedora-burn-yubikey script like normal.
To enroll using FreeOTP or Google Authenticator, go to
https://admin.fedoraproject.org/totpcgiprovision

What's enough authentication?
=============================
FAS Password+FreeOTP or FAS Password+Yubikey
Note: don't actually enter a +, simple enter your FAS Password and press your 
yubikey or enter your FreeOTP code. 

---------------------------------------------
Administrating and troubleshooting two factor
---------------------------------------------

Two factor auth is implemented by a modified copy of the
https://github.com/mricon/totp-cgi project doing the authentication and
pam_url submitting the authentication tokens.

totp-cgi runs on the fas servers (currently fas01.stg and fas01/fas02/fas03 in
production), listening on port 8443 for pam_url requests.

FreeOTP, Google authenticator and yubikeys are supported as tokens to use with
your password.

FreeOTP, Google authenticator:
==============================

FreeOTP application is preferred, however Google authenticator works as well.
(Note that Google authenticator is not open source)

This is handled via totpcgi. There's a command line tool to manage users, 
totpprov. See 'man totpprov' for more info. Admins can use this tool to revoke 
lost tokens (google authenticator only) with 'totpprov delete-user username' 

To enroll using FreeOTP or Google Authenticator for production machines, go to
https://admin.fedoraproject.org/totpcgiprovision

To enroll using FreeOTP or Google Authenticator for staging machines, go to
https://admin.stg.fedoraproject.org/totpcgiprovision/

You'll be prompted to login with your fas username and password.

Note that staging and production differ.

YubiKeys:
=========

Yubikeys are enrolled and managed in FAS. Users can self-enroll using the
fedora-burn-yubikey utility included in the fedora-packager package.

What do I do if I lose my token?
================================
Send an email to admin@fedoraproject.org that is encrypted/signed with your 
gpg key from FAS, or otherwise identifies you are you.

How to remove a token (so the user can re-enroll)?
==================================================
First we MUST verify that the user is who they say they are, using any of the
following: 

- Personal contact where the person can be verified by member of
  sysadmin-main. 

- Correct answers to security questions. 

- Email request to admin@fedoraproject.org that is gpg encrypted by the key
  listed for the user in fas. 

Then: 

1. For google authenticator, login to one of the fas machines and run: 
sudo totpprov delete-user username

2. For yubikey: login to one of the fas machines and run: 
/usr/local/bin/yubikey-remove.py username

The user can then go to https://admin.fedoraproject.org/totpcgiprovision/
and reprovision a new device. 

If the user emails admin@fedoraproject.org with the signed request, make sure
to reply to all indicating that a reset was performed.  This is so that other
admins don't step in and reset it again after its been reset once.
