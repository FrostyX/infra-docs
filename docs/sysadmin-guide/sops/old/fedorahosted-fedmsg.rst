.. title: Fedmsg Fedorahosted SOP
.. slug: infra-fedorahosted-fedmsg
.. date: 2013-08-21
.. taxonomy: Contributors/Infrastructure

======================================
Fedorahosted FedMsg Infrastructure SOP
======================================

Publish fedmsg messages from Fedora Hosted trac instances.

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-apps #fedora-admin, sysadmin-hosted
Location
	 Serverbeach
Servers
	 hosted03, hosted04
Purpose
	 Broadcast trac activity for select projects (opt-in)

Description
===========

fedmsg activity is usually an all-or-nothing proposition.  We emit messages
for all koji jobs and all bodhi updates, or none.

fedmsg activity for Fedora Hosted is another story.  We provide the option
for project owners to opt-in to fedmsg and have their activity broadcast,
but it is off by default.

This document describes how to:

1. Enable the fedmsg plugin for a fedora hosted project.
2. Setup the fedmsg plugin on a new node.

Enable the fedmsg plugin for a fedora hosted project.
=====================================================

Enable the trac plugin
----------------------

The trac-fedmsg-plugin package should be installed, but disabled.

Edit ``/srv/web/trac/projects/$PROJECT/conf/trac.ini``. Under the [components] section add::

  trac_fedmsg_plugin.* = enabled

And restart apache with "sudo apachectl graceful"

Enable the git hook
-------------------

There is an ansible playbook that does this.  There is no
need to do it by hand anymore.  Run::

  $ sudo -i ansible-playbook \
    /srv/web/infra/ansible/playbooks/fedorahosted_fedmsg_git.yml \
    --extra-vars '{"repos":["yanex.git"]}'


Enabling by hand
`````````````````

*If* you were to do it by hand, without the playbook, you could follow
the instructions below: Make a backup of the old post-receive hook.  It
should be empty when you encounter it, but just to be safe::

  $ mv /srv/git/$PROJECT.git/hooks/post-receive \
    /srv/git/$PROJECT.git/hooks/post-receive.orig

Then, symlink in the new post-receive hook with::

  $ ln -s /usr/local/share/git/hooks/post-receive-fedorahosted-fedmsg \
    /srv/git/$PROJECT.git/hooks/post-receive

That hooks is managed by ansible -- if you want to modify it you can do
so there.

.. note:: IF there was an old post-receive hook in place, you should
          check to see if it did something important.  The 'fedora-web' git
          repo (which was converted early on) had such a hook.  See
          /srv/git/fedora-web.git/hooks for an example of how to handle
          multiple git hooks.  Something like
          /usr/share/git-core/post-receive-chained can be used to chain the
          hook across multiple scripts.


How to setup the fedmsg plugin on a new fedorahosted node.
==========================================================

1) Create certs for the new node as per the fedmsg-certs doc.

2) Declare those certs in `/etc/fedmsg.d/ssl.py`` globally.

3) Declare endpoints for the new node in ``/etc/fedmsg.d/endpoints.py``.

4) Use our configuration management tool to distribute that new global
    fedmsg config to the new node and all other nodes.

5) Install the trac-fedmsg-plugin package on the new node and follow the
    steps above.
