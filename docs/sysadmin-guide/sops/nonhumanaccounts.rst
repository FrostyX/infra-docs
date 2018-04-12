.. title: Non-human Accounts Infrastructure SOP
.. slug: infra-nonhuman-accounts
.. date: 2018-03-14
.. taxonomy: Contributors/Infrastructure

=====================================
Non-human Accounts Infrastructure SOP
=====================================

We have many non-human accounts for various services, used by our web
applications and certain automated scripts.

Contents
========

1. Contact Information
2. FAS Accounts
3. Bugzilla Accounts
4. PackageDB Owners
5. Koji Accounts

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin
Persons: 
  sysadmin-main
Purpose: 
  Provide Non-human accounts to our various services

Tokens
======
Wherever possible OIDC (Open Id Connect) tokens or other tokens should be 
used for the script. Whatever the token it should have the minimum privs to 
do whatever the script or process needs to do and no more. 

Depending on what service(s) it needs to interact with this could different 
tokens. Consult with the Fedora Security Officer for exact details. 
