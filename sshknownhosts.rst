.. title: SSH Known Hosts Infrastructure SOP
.. slug: infra-ssh-known-hosts
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

==================================
SSH known hosts Infrastructure SOP
==================================

Provides Known Hosts file that is globally deployed and publicly available at
https://admin.fedoraproject.org/ssh_known_hosts

Contact Information
===================
Owner:
  Fedora Infrastructure Team
Contact:
  #fedora-admin, sysadmin group
Location:
  all
Servers:
  all
Purpose:
  Provides Known Hosts file that is globally deployed.


Adding  a host alias to the ssh_known_hosts
===========================================

If you need to add a host alias to a host in ssh_known_hosts simply
go to the dir for the host in infra-hosts and add a file named host_aliases
to the git repo in that dir. Put one alias per line and save.

Then the next time fetch-ssh-keys runs it will add those aliases to known hosts.
