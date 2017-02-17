.. title: fedmsg Notifications SOP
.. slug: infra-fmn
.. date: 2015-03-24
.. taxonomy: Contributors/Infrastructure

==============================
FedMsg Notifications (FMN) SOP
==============================

Route individualized notifications to fedora contributors over email, irc.


Contact Information
===================

Owner
-----

- Messaging SIG
- Fedora Infrastructure Team

Contact
-------

  - #fedora-apps for FMN development

  - #fedora-fedmsg for an IRC feed of all fedmsgs

  - #fedora-admin for problems with the deployment of FMN

  - #fedora-noc for outage/crisis alerts

Servers
-------

Production servers:

  - notifs-backend01.phx2.fedoraproject.org (RHEL 7)
  - notifs-web01.phx2.fedoraproject.org (RHEL 7)
  - notifs-web02.phx2.fedoraproject.org (RHEL 7)

Staging servers:

  - notifs-backend01.stg.phx2.fedoraproject.org (RHEL 7)
  - notifs-web01.stg.phx2.fedoraproject.org (RHEL 7)
  - notifs-web02.stg.phx2.fedoraproject.org (RHEL 7)

Purpose
-------

Route notifications to users


Description
===========

fmn is a pair of systems intended to route fedmsg notifications to Fedora
contributors and users.

There is a web interface running on notifs-web01 and notifs-web02 that
allows users to login and configure their preferences to select this or that
type of message.

There is a backend running on notifs-backend01 where most of the work is
done.

The backend process is a 'fedmsg-hub' daemon, controlled by systemd.


Hosts
=====

notifs-backend
--------------

This host runs:

- ``fedmsg-hub.service``

- One or more ``fmn-worker@.service``. Currently notifs-backend01 runs
  ``fmn-worker@{1-4}.service``

- ``fmn-backend@1.service``

- ``fmn-digests@1.service``

- ``rabbitmq-server.service``, an AMQP broker used to communicate between the services.

- ``redis.service``, used for caching.

This host relies on a PostgreSQL database running on db01.phx2.fedoraproject.org.

notifs-web
----------

This host runs:

- A Python WSGI application via Apache httpd that serves the `FMN web user interface`_.

This host relies on a PostgreSQL database running on db01.phx2.fedoraproject.org.


Deployment
==========

Once upstream releases a new version of `fmn`_, `fmn-web`_, or `fmn-sse`_
creating a Git tag, a new version can be built an deployed into Fedora infrastructure.

Building
--------

FMN is packaged in Fedora and EPEL as `python-fmn`_ (the backend), `python-fmn-web`_
(the frontend), and the optional `python-fmn-sse`_.

Since all the hosts run RHEL 7, you need to build all these packages for EPEL 7.

Configuration
-------------

If there are any configuration updates required by the new version of FMN, update the
``notifs`` Ansible roles on batcave01.phx2.fedoraproject.org. Remember to use::

    {% if env == 'staging' %}
        <new config here>
    {% else %}
        <retain old config>
    {% endif %}

When deploying the update to staging. You can apply configuration updates to staging
by running::

    $ sudo rbac-playbook -l staging groups/notifs-backend.yml
    $ sudo rbac-playbook -l staging groups/notifs-web.yml

Simply drop the ``-l staging`` to update the production configuration.

Upgrading
---------

To upgrade the `python-fmn`_, `python-fmn-web`_, and `python-fmn-sse`_ packages, apply
configuration changes, and restart the services, you should use the manual upgrade
playbook::

    $ sudo rbac-playbook -l staging manual/upgrade/fmn.yml

Again, drop the ``-l staging`` flag to upgrade production.

Be aware that the FMN services take a significant amount of time to start up as they
pre-heat their caches before starting work.


Service Administration
======================

Disable an account (on notifs-backend01)::

  $ sudo -u fedmsg /usr/local/bin/fmn-disable-account USERNAME

Restart::

  $ sudo systemctl restart fedmsg-hub

Watch logs::

  $ sudo journalctl -u fedmsg-hub -f

Configuration::

  $ ls /etc/fedmsg.d/
  $ sudo fedmsg-config | less

Monitor performance::
  
  http://threebean.org/fedmsg-health-day.html#FMN

Upgrade (from batcave)::

  $ sudo -i ansible-playbook /srv/web/infra/ansible/playbooks/manual/upgrade/fmn.yml

Mailing Lists
-------------

We use FMN as a way to forward certain kinds of messages to mailing lists so
people can read them the good old fashioned way that they like to.  To
accomplish this, we create 'bot' FAS accounts with their own FMN profiles and
we set their email addresses to the lists in question.

If you need to change the way some set of messages are forwarded, you can do
it from the FMN web interface (if you are an FMN admin as defined in the config
file in roles/notifs/frontend/).  You can navigate to
https://apps.fedoraproject.org/notifications/USERNAME.id.fedoraproject.org to do
this.

If the account exists as a FAS user already (for instance, the ``virtmaint``
user) but it does not yet exist in FMN, you can add it to the FMN database by
logging in to notifs-backend01 and running ``fmn-create-user --email
DESTINATION@EMAIL.COM --create-defaults FAS_USERNAME``.


.. _fmn: https://github.com/fedora-infra/fmn
.. _fmn-web: https://github.com/fedora-infra/fmn.web
.. _fmn-sse: https://github.com/fedora-infra/fmn.sse
.. _python-fmn: https://admin.fedoraproject.org/pkgdb/package/rpms/python-fmn/
.. _python-fmn-web: https://admin.fedoraproject.org/pkgdb/package/rpms/python-fmn-web/
.. _python-fmn-sse: https://admin.fedoraproject.org/pkgdb/package/rpms/python-fmn-sse/
.. _FMN web user interface: https://apps.fedoraproject.org/notifications>
