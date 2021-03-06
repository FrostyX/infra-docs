.. title: SELinux Infrastructure SOP
.. slug: infra-selinux
.. date: 2012-03-19
.. taxonomy: Contributors/Infrastructure

==========================
SELinux Infrastructure SOP
==========================

SELinux is a fundamental part of our Operating System but still has a
large learning curve and remains quite intimidating to both developers and
system administrators. Fedora's Infrastructure has been growing at an
unfathomable rate, and is full of custom software that needs to be locked
down. The goal of this SOP is to make it simple to track down and fix
SELinux policy related issues within Fedora's Infrastructure.

Fully deploying SELinux is still an ongoing task, and can be tracked in
fedora-infrastructure [45]ticket #230.

Contents
========

1. Contact Information
2. Step One: Realizing you have a problem
3. Step Two: Tracking down the violation
4. Step Three: Fixing the violation

  1. Allowing ports
  2. Toggling an SELinux boolean
  3. Setting custom context
  4. Deploying custom policy modules

Contact Information
===================

Owner
  Fedora Infrastructure Team
Contact
  #fedora-admin, sysadmin-main, sysadmin-web groups
Purpose
  To ensure that we are able to fully wield the power of SELinux
  within our infrastructure.

Step One: Realizing you have a problem
=======================================

If you are trying to find a specific problem on a host go look in the
audit.log per-host on our cental log server. See the syslog SOP for
more information.

Step Two: Tracking down the violation
=====================================

Generate SELinux policy allow rules from logs of denied operations. This
is useful for getting a quick overview of what has been getting denied on
the local machine::
      
  audit2allow -la

You can obtain more detailed audit messages by using ausearch to get the
most recent violations::

  ausearch -m avc -ts recent

Again -see the syslog SOP for more information here.

Step Three: Fixing the violation
================================

Below are examples of using our current ansible configuration to make
SELinux deployment changes. These constructs are currently home-brewed,
and do not exist in upstream Ansible. For these functions to work, you must
ensure that the host or servergroup is configured with 'include selinux',
which will enable SELinux in permissive mode. Once a host is properly
configured, this can be changed to 'include selinux-enforcing' to enable
SELinux Enforcing mode.

.. note::
  Most services have $service_selinux manpages that are automatically generated from policy.

Toggling an SELinux boolean
---------------------------

SELinux booleans, which can be viewed by running `semanage boolean -l`,
can easily be configured using the following syntax within your ansible
configuration.::

  seboolean: name=httpd_can_network_connect_db state=yes persistent=yes

Setting custom context
----------------------

Our infrastructure contains many custom applications, which may utilize
non-standard file locations. These issues can lead to trouble with
SELinux, but they can easily be resolved by setting custom file context.::

  "file: path=/var/tmp/l10n-data recurse=yes setype=httpd_sys_content_t"


Fixing odd errors from the logs
-------------------------------
If you see messages like this in the log reports::

  restorecon:/etc/selinux/targeted/contexts/files/file_contexts: Multiple same / specifications for /home/fedora.
  matchpathcon: / /etc/selinux/targeted/contexts/files/file_contexts: Multiple same / / specifications for /home/fedora.

Then it is likely you have an overlapping filecontext in your local selinux context configuration - in this case likely one added by ansible accidentally.

To find it run this::

  semanage fcontext -l | grep /path/being/complained/about

sometimes it is just an ordering problem and reversing them solves it
other times it is just an overlap, period.

look at the context and delete the one you do not want or reorder.

To delete run::     

  semanage fcontext -d '/entry/you/wish/to/delete'

This just removes that filecontext - no need to worry about files being deleted.

Then rerun the triggering command and see if the problem is solved.
