.. title: Wiki Infrastructure SOP
.. slug: infra-wiki
.. date: 2012-09-13
.. taxonomy: Contributors/Infrastructure

=======================
Wiki Infrastructure SOP
=======================

   Managing our wiki.

Contact Information
===================

Owner
	 Fedora Infrastructure Team / Fedora Website Team
Contact
	 #fedora-admin or #fedora-websites on irc.freenode.net
Location: http
	//fedoraproject.org/wiki/
Servers
	 proxy[1-3] app[1-2,4]
Purpose
	 Provides our production wiki

Description
===========
Our wiki currently runs mediawiki.

.. important::
  Whenever you changes anything on the wiki (bugfix, configuration, plugins,
  ...), please update the page at https://fedoraproject.org/wiki/WikiChanges .

Dealing with Spammers:
=======================
If you find a spammer is editing pages in the wiki do the following:

1. admin disable their account in fas, add 'wiki spammer' as the comment
2. block their account in the wiki from editing any additional pages 
3. go to the list of pages they've edited and rollback their changes - one by one. If there are many get someone to help you.

