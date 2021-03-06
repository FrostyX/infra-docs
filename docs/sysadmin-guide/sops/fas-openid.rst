.. title: FAS-OpenID
.. slug: infra-fas-openid
.. date: 2013-12-14
.. taxonomy: Contributors/Infrastructure

==========
FAS-OpenID
==========


FAS-OpenID is the OpenID server of Fedora infrastructure.

Live instance is at https://id.fedoraproject.org/
Staging instance is at https://id.dev.fedoraproject.org/

Contact Information
===================

Owner
  Patrick Uiterwijk (puiterwijk)
Contact
  #fedora-admin, #fedora-apps, #fedora-noc
Location
  openid0{1,2}.phx2.fedoraproject.org
  openid01.stg.fedoraproject.org
Purpose
  Authentication & Authorization

Trusted roots
==============

FAS-OpenID has a set of "trusted roots", which contains websites which are
always trusted, and thus FAS-OpenID will not show the Approve/Reject form to
the user when they login to any such site.
   
As a policy, we will only add websites to this list which Fedora
Infrastructure controls. If anyone ever ask to add a website to this list,
just answer with this default message::

  We only add websites we (Fedora Infrastructure) maintain to this list.

  This feature was put in because it wouldn't make sense to ask for permission
  to send data to the same set of servers that it already came from.

  Also, if we were to add external websites, we would need to judge their
  privacy policy etc.

  Also, people might start complaining that we added site X but not their site,
  maybe causing us "political" issues later down the road.

  As a result, we do NOT add external websites.

