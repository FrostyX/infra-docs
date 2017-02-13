.. title: gather-easyfix SOP
.. slug: infra-gather-easyfix
.. date: 2016-03-14
.. taxonomy: Contributors/Infrastructure

=========================
Fedora gather easyfix SOP
=========================

Fedora-gather-easyfix as the name says gather tickets marked as easyfix from
multiple sources (pagure, github and fedorahosted currently). Providing a single
place for new-comers to find small tasks to work on.


Contents
========

1. Contact Information
2. Documentation Links

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	http://fedoraproject.org/easyfix/
Servers
    sundries01, sundries02, sundries01.stg
Purpose
    Gather easyfix tickets from multiple sources.


Upstream sources are hosted on github at:
https://github.com/fedora-infra/fedora-gather-easyfix/

The files are then mirrored to our ansible repo, under the `easyfix/gather`
role.

The project is a simple script ``gather_easyfix.py`` gathering information from
the projects sets on the `Fedora wiki
<https://fedoraproject.org/wiki/Easyfix>`_ and outputing a single html file.
This html file is then improved via the css and javascript files present in the
sources.

The generated html file together with the css and js files are then synced to
the proxies for public consumption :)
