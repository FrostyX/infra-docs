.. title: Link Tracking SOP
.. slug: infra-link-tracking
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

=============
Link tracking
=============

Using link tracking is [43]an easy way for us to find out how people are
getting to our download page. People might click over to our download page
from any of a number of areas, and knowing the relative usage of those
links can help us understand what materials we're producing are more
effective than others.

Adding links
============

Each link should be constructed by adding ? to the URL, followed by a
short code that includes:

* an indicator for the link source (such as the wiki release notes)
* an indicator for the Fedora release in specific (such as F15 for the
  final, or F15a for the Alpha test release)

So a link to get.fp.o from the one-page release notes would become
http://get.fedoraproject.org/?opF15.

FAQ
===
I want to copy a link to my status update for social networking, or my blog.
  If you're posting a status update to identi.ca, for example, use
  the link tracking code for status updates. Don't copy a link
  straight from an announcement that includes link tracking from the
  announcement. You can copy the link itself but remember to change
  the portion after the ? to instead use the st code for status
  updates and blogs, followed by the Fedora release version (such as
  F16a, F16b, or F16), like this::

    http://fedoraproject.org/get-prerelease?stF16a

I want to point people to the announcement from my blog. Should I use the announcement link tracking code?
  The actual URL link itself is the announcement URL. Add the link
  tracking code for blogs, which would start with ?st and end with
  the Fedora release version, like this::

    http://fedoraproject.org/wiki/F16_release_announcement?stF16a

The codes
=========

.. note::
   Additions to this table are welcome.

=============================================== ==========
Link source                                     Code
=============================================== ==========
Email announcements                             an
----------------------------------------------- ----------
Wiki announcements                              wkan
----------------------------------------------- ----------
Front page                                      fp
----------------------------------------------- ----------
Front page of wiki                              wkfp
----------------------------------------------- ----------
The press release Red Hat makes                 rhpr
----------------------------------------------- ----------
http://redhat.com/fedora                        rhf
----------------------------------------------- ----------
Test phase release notes on                     wkrn
----------------------------------------------- ----------
Official release notes                          rn
----------------------------------------------- ----------
Official installation guide                     ig
----------------------------------------------- ----------
One-page release notes                          op
----------------------------------------------- ----------
Status links (blogs, social media)              st
=============================================== ==========

