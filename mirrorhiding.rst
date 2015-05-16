.. title: Mirror Hiding Infrastructure SOP
.. slug: infra-mirror-hiding
.. date: 2011-08-23
.. taxonomy: Contributors/Infrastructure

================================
Mirror hiding Infrastructure SOP
================================

At times, such as release day, there may be a conflict between Red Hat
trying to release content for RHEL, and Fedora trying to release Fedora.
One way to limit the pain to Red Hat on release day is to hide
download.fedora.redhat.com from the publiclist and mirrorlist redirector,
which will keep most people from downloading the content from Red Hat
directly.

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main, sysadmin-web group
Location
	 Phoenix
Servers
	 app3, app4
Purpose
	 Hide Public Mirrors from the publiclist / mirrorlist redirector

Description
===========
To hide a public mirror, so it doesn't appear on the publiclist or the
mirrorlist, simply go into the MirrorManager administrative web user
interface, at [45]https://admin.fedoraproject.org/mirrormanager. Fedora
sysadmins can see all Sites and Hosts. For each Site and Host, there is a
checkbox marked "private", which if set, will hide that Site (and all its
Hosts), or just that single Host, such that it won't appear on the public
lists.

To make a private-marked mirror public, simply clear the "private"
checkbox again.

This change takes effect at the top of each hour.

