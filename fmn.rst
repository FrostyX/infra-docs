.. title: fedmsg Notifications SOP
.. slug: infra-fmn
.. date: 2015-03-24
.. taxonomy: Contributors/Infrastructure

==============================
fmn (fedmsg notifications) SOP
==============================

Route individualized notifications to fedora contributors over email, irc.

Contact Information
-------------------

Owner
	Messaging SIG, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-fedmsg, #fedora-admin, #fedora-noc
Servers
	notifs-backend01, notifs-web0{1,2}
Purpose
	Route notifications to users

Description
-----------

fmn is a pair of systems intended to route fedmsg notifications to Fedora
contributors and users.

There is a web interface running on notifs-web01 and notifs-web02 that
allows users to login and configure their preferences to select this or that
type of message.  

There is a backend running on notifs-backend01 where most of the work is
done.

The backend process is a 'fedmsg-hub' daemon, controlled by systemd.

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
