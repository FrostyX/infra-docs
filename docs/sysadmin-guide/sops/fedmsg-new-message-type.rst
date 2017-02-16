.. title: Adding a new fedmsg message type
.. slug: fedmsg-new-message-type
.. date: 2016-05-27

================================
Adding a new fedmsg message type
================================


Instrumenting the program
=========================
First, figure out how you're going to publish the message?  Is it from a shell
script or from a long running process?

If its from shell script, you need to just add a `fedmsg-logger` statement to
the script.  Remember to set the `--modname` and `--topic` for your new
message's fully-qualified topic.

If its from a python process, you need to just add a ``fedmsg.publish(..)``
call.  The same concerns about modname and topic apply here.

If this is a short-lived python process, you'll want to add `active=True` to the
call to ``fedmsg.publish(..)``.  This will make the fedmsg lib "actively" reach
out to our fedmsg-relay running on busgateway01.

If it is a long-running python process (like a WSGI thread), then you don't need
to pass any extra arguments.  You don't want it to reach out to the fedmsg-relay
if possible.  Your process will require that some "endpoints" are created for it
in ``/etc/fedmsg.d/``.  More on that below.

Supporting infrastructure
=========================


You need to make sure that the machine this is running on has a cert and key
that can be read by the program to sign its message.  If you don't have a cert
already, then you need to create it in the private repo.  Ask a sysadmin-main
member.

Then you need to declare those certs in the `fedmsg_certs` data structure stored
typically in our ansible ``group_vars/`` for this service.  Declare both the
name of the cert, what group and user it should be owned by, and in the
``can_send:`` section, declare the list of topics that this cert should be
allowed to publish.

If this is a long-running python process that is *not* passing `active=True` to
the call to `fedmsg.publish(..)`, then you have to also declare endpoints for
it.  You do that by specifying the ``fedmsg_wsgi_procs`` and
``fedmsg_wsgi_vars`` in the ``group_vars`` for your service.  The iptables rules
and fedmsg endpoints should be automatically created for you on the next
playbook run.

Supporting code
===============

At this point, you can push the change out to production and be publishing
messages "okay".  Everything should be fine.

However, your message will show up blank in datagrepper, in IRC, and in FMN, and
everywhere else we try to render it.  You *must* then follow up and write a new
`Processor` for it in the fedmsg_meta library we maintain:
https://github.com/fedora-infra/fedmsg_meta_fedora_infrastructure

You also *must* write a test case for it there.  The docs listing all topics we
publish at http://fedora-fedmsg.rtfd.org/ is automatically generated from the
test suite.  Please don't forget this.

Lastly, you should cut a release of fedmsg_meta and deploy it using the
`playbooks/manual/upgrade/fedmsg.yml` playbook, which should update all the
relevant hosts.

Corner cases
============

If the process publishing the new message lives *outside* our main network, you
have to jump through more hoops.  Look at abrt, koschei, and copr for examples
of how to configure this (you need a special firewall rule, and they need to be
configured to talk to our "inbound gateway" running on the proxies.
