.. title: Content Hosting Infrastructure SOP
.. slug: infra-content-hosting
.. date: 2012-07-17
.. taxonomy: Contributors/Infrastructure

==================================
Content Hosting Infrastructure SOP
==================================

Contact Information
===================

Owner
  Fedora Infrastructure Team

Contact
  #fedora-admin, sysadmin-main, fedora-infrastructure-list

Location
  Phoenix

Servers
  secondary1, netapp[1-3], torrent1

Purpose
  Policy regarding hosting, removal and pruning of content.

Scope
  download.fedora.redhat.com, alt.fedoraproject.org,
  archives.fedoraproject.org, secondary.fedoraproject.org,
  torrent.fedoraproject.org

Description
===========

Fedora hosts both Fedora content and some non-Fedora content. Our
resources are finite and as such we have to have some policy around when
to remove old content. This SOP describes the test to remove content. The
spirit of this SOP is to allow more people to host content and give it a
try, prove that it's useful. If it's not popular or useful, it will get
removed. Also out of date or expired content will be removed.

What hosting options are available
----------------------------------

Aside from the hosting at https://pagure.io/ we have a series of
mirrors we're allowing people to use. They are located at:

* http://archive.fedoraproject.org/pub/archive/ - For archives of historical Fedora releases
* http://secondary.fedoraproject.org/pub/fedora-secondary/ - For secondary architectures
* http://alt.fedoraproject.org/pub/alt/ - For misc content / catchall
* http://torrent.fedoraproject.org/ - For torrent hosting
* http://spins.fedoraproject.org/ - For official Fedora Spins hosting, mirrored somewhat
* http://download.fedoraproject.com/pub/ - For official Fedora Releases, mirrored widely

Who can host? What can be hosted?
---------------------------------
Any official Fedora content can hosted and made available for mirroring.
Official content is determined by the Council by virtue of allowing people
to use the Fedora trademark. People representing these teams will be
allowed to host.

Non Official Hosting
--------------------

People wanting to host unofficial bits may request approval for hosting.
Create a ticket at https://pagure.io/fedora-infrastructure/
explaining what and why Fedora should host it. Such will be reviewed by
the Fedora Infrastructure team.

Requests for non-official hosting that may conflict with existing Fedora
policies will be escalated to the Council for approval.

Licensing
---------
Anything hosted with Fedora must come with a Free software license that is
approved by Fedora. See http://fedoraproject.org/wiki/Licensing for
more.

Requesting Space
================

* Make sure you have a Fedora account -
  https://admin.fedoraproject.org/accounts/
* Ensure you have signed the Fedora Project Contributor Agreement (FPCA)
* Submit a hosting request -
  https://pagure.io/fedora-infrastructure/

  * Include who you are, and any group you are working with (e.g. a SIG)
  * Include Space requirements
  * Include an estimate of the number of downloads expected (if you can).
  * Include the nature of the bits you want to host.

* Apply for group hosted-content -
  https://admin.fedoraproject.org/accounts/group/view/hosted-content

Using Space
===========

A dedicated namespace in the mirror will be assigned to you. It will be
your responsibility to upload content, remove old content, stay within
your quota, etc. If you have any questions or concerns about this please
let us know. Generally you will use rsync. For example::

  rsync -av --progress ./my.iso secondary01.fedoraproject.org:/srv/pub/alt/mySpace/

.. important:: 
  None of our mirrored content is backed up. Ensure that you keep backups of
  your content.

Content Pruning / Purging / Removal
===================================

The following guidelines / tests will be used to determine whether or not
to remove content from the mirror.

Expired / Old Content
----------------------

If content meets any of the following criteria it may be removed:

* Content that has reached the end of life (is no longer receiving  updates).
* Pre-release content that has been superceded.
* EOL releases that have been moved to archives.
* N-2 or greater releases. If more than 3 versions of a piece of content
  are on the mirror, the oldest may be removed.

Limited Use Content
-------------------
If content meets any of the following criteria it may be removed:

* Content with exceedingly limited seeders or downloaders, with little
  prospect of increasing those numbers and which is older then 1 year.
 
* Content such as videos or audio which are several years old.

Catch All Removal
------------------

Fedora reserves the right to remove any content for any reason at any
time. We'll do our best to host things but sometimes we'll need space or
just need to remove stuff for legal or policy reasons.
