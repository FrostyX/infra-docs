.. title: github2fedmsg SOP
.. slug: infra-github2fedmsg
.. date: 2016-04-08
.. taxonomy: Contributors/Infrastructure

=================
github2fedmsg SOP
=================

Bridge github events onto our fedmsg bus.

App:     https://apps.fedoraproject.org/github2fedmsg/
Source:  https://github.com/fedora-infra/github2fedmsg/

Contact Information
===================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-admin, #fedora-noc
Servers
	github2fedmsg01
Purpose
    Bridge github events onto our fedmsg bus.

Description
===========

github2fedmsg is a small Python Pyramid app that bridges github events onto our
fedmsg bus by way of github's "webhooks" feature.  It is what allows us to have
IRC notifications of github activity via fedmsg.  It has two phases of
operation:

- Infrequently, a user will log in to github2fedmsg via Fedora OpenID.  They
  then push a button to also log in to github.com.  They are then logged in to
  github2fedmsg with *both* their FAS account and their github account.

  They are then presented with a list of their github repositories.  They can
  toggle each one: "on" or "off".  When they turn a repo on, our webapp makes a
  request to github.com to install a "webhook" for that repo with a callback URL
  to our app.

- When events happen to that repo on github.com, github looks up our callback
  URL and makes an http POST request to us, informing us of the event.  Our
  github2fedmsg app receives that, validates it, and then republishes the
  content to our fedmsg bus.

What could go wrong?
====================

- Restarting the app or rebooting the host shouldn't cause a problem.  It should
  come right back up.

- Our database could die.  We have a db with a list of all the repos we have
  turned on and off.  We would want to restore that from backup.

- If github gets compromised, they might have to revoke all of their application
  credentials.  In that case, our app would fail to work.  There are *lots* of
  private secrets set in our private repo that allow our app to talk to
  github.com.  There are inline comments there with instructions about how to
  generate new keys and secrets.
