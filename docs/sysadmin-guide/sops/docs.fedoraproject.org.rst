.. title: Fedra Documentation SOP
.. slug: Fedora-docs
.. date: 2017-03-24
.. taxonomy: Contributors/Infrastructure

========
docs SOP
========

  Fedora Documentation - Documentation for installing and using Fedora

Contact Information
-------------------

Owner:
  docs, Fedora Infrastrcture Team
Contact:
  #fedora-docs
Servers:
  proxy*
Purpose:
  Provide documentation for users and contributors.

Description:
------------

The Fedora Documentation Project was created to provide documentation 
for fedora users and contributors. It's like "The Bible" for using Fedora
and other software used by the Fedora Project. It uses Publican, a free
and open-source publishing tool. Publican generates html pages from content
in DocBook XML format. The source files are in a git repo and publican
builds html files from these source files whenever changes are made.
As these are static pages these are available on all the proxy servers
which serve our requests for docs.fedoraproject.org.

Updates process:
----------------
The fedora docs writers update and build their docs and then push
the completed output into a git repo. This git repo is then pulled
by each of the Fedora proxies and served as static content. 

Note that docs is talking about setting up a new process,
this SOP needs updating when that happens. 

Reporting bugs:
---------------
Bugs can be reported at the Fedora Documentation's Bugzilla. 
Here's the link:
https://bugzilla.redhat.com/enter_bug.cgi?product=Fedora%20Documentation

Errors or problems in the wiki can be modified by anyone with a FAS account.

Contributing to the Fedora Documentation Project:
-------------------------------------------------

If you find the existing documentation insufficient or outdated
or any particular page is not available in your language feel free
to improve the documentation by contributing to Fedora Documentation Project. 
You can find more details here: https://fedoraproject.org/wiki/Join_the_Docs_Project

Translation of documentation is taken care by the Fedora Localization
Project aka L10N.
More details can be found at: https://fedoraproject.org/wiki/L10N

Publican wiki:
--------------

More details about Publican can be found at the publican wiki here:
https://sourceware.org/publican/en-US/index.html 
 
