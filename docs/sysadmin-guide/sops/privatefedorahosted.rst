.. title: Fedorahosted Private Tickets SOP
.. slug: infra-fedorahosted-private-tickets
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

===============================================
Private fedorahosted tickets Infrastructure SOP
===============================================

Provides for users only viewing tickets they are involved with.

Contact Information
===================

Owner
	Fedora Infrastructure Team

Contact
	#fedora-admin, sysadmin-hosted

Location
	<not sure what to put here>

Servers
	hosted1

Purpose
	Provides for users only viewing tickets they are involved with.

Description
===========

Fedora Hosted Projects have the option of setting ticket permissions so
that only users involved with tickets can see them. This plugin requires
someone in sysadmin-hosted to set it up, and requires justification to
use. The only current implementation is a request tracking system at
[45]https://fedorahosted.org/famnarequests for tracking requests for North
American ambassadors since mailing addresses, etc will be put in there.

Implementation
==============

On hosted1::

  sudo -u apache vim /srv/web/trac/projects/<project name>/conf/trac.ini

Add the following to the appropriate sections of ``trac.ini``::

   [privatetickets]
   group_blacklist = anonymous, authenticated

   [components]
   privatetickets.* = enabled

   [trac]
   permission_policies = PrivateTicketsPolicy, DefaultPermissionPolicy, LegacyAttachmentPolicy

.. note:: For projects not currently using plugins, you'll have to add the
  [components] section, and you'll need to add the permission_policies to
  the [trac] section.

Next, someone with TRAC_ADMIN needs to grant TICKET_VIEW_SELF (a new
permission) to authenticated. This permission allows users to view tickets
that they are either owner, CC, or reporter on. There are other options
more fully described at [46]the upstream site.

Make sure that TICKET_VIEW is removed from anonymous, or else this plugin
will have no effect.

