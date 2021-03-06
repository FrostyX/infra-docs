.. title: Infrastucture Orientation SOP
.. slug: infra-orientation
.. date: 2016-10-20
.. taxonomy: Contributors/Infrastructure

==============================
Orientation Infrastructure SOP
==============================

Basic orientation and introduction to the sysadmin group. Welcome aboard!

Contents
========

1. Contact Information
2. Description
3. Welcome to the team

  1. Time commitment
  2. Prove Yourself

4. Doing Work

  1. Ansible

5. Our Setup
6. Our Rules

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin, sysadmin-main
Purpose
	 Provide basic orientation and introduction to the sysadmin group

Description
===========

Fedora's Infrastructure team is charged with keeping all the lights on,
improving pain points, expanding services, designing new services and
partnering with other teams to help with their needs. The team is highly
dynamic and primarily based in the US. This is only significant in that
most of us work during the day in US time. We do have team members all
over the globe though and generally have decent coverage. If you happen to
be one of those who is not in a traditional US time zone you are
encouraged to be around, especially in #fedora-admin during those times
when we have less coverage. Even if it is just to say "I can't help with
that but $ADMIN will be and he should be here in about 3 hours".

The team itself is generally friendly and honest. Don't be afraid to
disagree with someone, even if you're new and they're an old timer. Just
make sure you ask yourself what is important to you and make sure to
provide data, we like that. We generally communicate on irc.freenode.net
in #fedora-admin. We have our weekly meetings on IRC and its the quickest
way to get in touch with everyone. Secondary to that we use the mailing
list. After that its our ticketing system and talk.fedoraproject.org.

*Welcome to the team!*

Time commitment
===============

Often times this is the biggest reason for turnover in our group. Some
groups like sysadmin-web and certainly sysadmin-main require a huge time
commitment. Don't be surprised if you see people working between 10-30
hours a week on various tasks and that's the volunteers. Your time
commitment is something personal to each individual and its something that
you should take some serious thought about. In general it's almost
impossible to be a regular part of the team without at least 5-10 hours a
week dedicated to the Infrastructure team.

Also note, if you are going to be away, let us know. As a volunteer we
can't possibly ask you to always be around all the time. Even if you're in
the middle of a project and have to stop, let us know. Nothing is worse
then thinking someone is working on something or will be around and
they're just not. Really, we all understand, got a test coming up? Busier
at work then normal? Going on a vacation? It doesn't matter, just let us
know when you're going to be gone and what you're working on so it doesn't
get forgotten.

Additionally don't forget that its worth it to discuss with your employer
about giving time during work. They may be all for it.

Prove Yourself
==============

This is one of the most difficult aspects of getting involved with our
team. We can't just give access to everyone who asks for it and often
actually doing work without access is difficult. Some of the best things
you can do are:

* Keep bugging people for work. It shows you're committed.
* Go through bugs, look at stale bugs and close bugs that have been fixed
* Try to duplicate bugs on your workstation and fix them there

Above all stick with it. Part of proving yourself is also to show the time
commitment it actually does take.

Doing Work
==========
Once you've been sponsored for a team its generally your job to find what
work needs to be done in the ticketing system. Be proactive about this.
The tickets can be found at:

https://pagure.io/fedora-infrastructure/issues

When you find a ticket that interests you contact your sponsor or the
ticket owner and offer help. While you're getting used to the way things
work, don't be offput by someone saying no or you can't work on that. It
happens, sometimes its a security thing, sometimes its a "I'm half way
through it and I'm not happy with where it is thing." Just move on to the
next ticket and go from there.

Also don't be surprised if some of the work involved includes testing on
your own workstation. Just setup a virtual environment and get to work!
There's a lot of work that can be done to prove yourself that involves no
access at all. Doing this kind of work is a sure fire way to get in to
more groups and get more involved. Don't be afraid to take on tasks you
don't already know how to do. But don't take on something you know you
won't be able to do. Ask for help when you need it and keep in contact
with your sponsor so you know

Ansible
=======

Things we do gets done in Ansible. It is important that you not make changes directly on
servers. This is for many reasons but just always make changes in
Ansible. If you want to get more familiar with Ansible, set it
up yourself and give it a try. The docs are available at
https://docs.ansible.com/

Our Setup
=========

Most of our work is done via bastion.fedoraproject.org. That host has
access to our other hosts, many of which are all over the globe. We have a
vpn solution setup so that knowing where the servers physically are is
only important when troubleshooting things. When you first get granted
access to one of the sysadmin-* groups, the first place you should turn is
bastion.fedoraproject.org then from there ssh to batcave01.

We also have an architecture repo available in our git repo. To get a copy
of this repo just::

  dnf install git
  git clone https://pagure.io/fedora-infrastructure.git

This will allow you to look through (and help fix) some of our scripts as
well as have access to our architectural documentation. Become familiar
with those docs if you're curious. There's always room to do better
documentation so if you're interested just ping your sponsor and ask about
it.

Our Rules
=========
The Fedora Infrastructure Team does have some rules. First is the security
policy. Please ensure you are compliant with:

https://infrastructure.fedoraproject.org/csi/security-policy/

before logging in to any of our servers. Many of those items rely on the
honor system.

Additionally note that any of the software we deploy must be available in
Fedora. There are some rare exceptions to this (particularly as it relates
to specific applications to Fedora). But each exception is taken on a case
by case basis.
