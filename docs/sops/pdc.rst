.. title: PDC SOP
.. slug: infra-pdc
.. date: 2016-04-07
.. taxonomy: Contributors/Infrastructure

=======
PDC SOP
=======

Store metadata about composes we produce and "component groups".

App:                  https://pdc.fedoraproject.org/
Source for frontend:  https://github.com/product-definition-center/product-definition-center
Source for backend:   https://github.com/fedora-infra/pdc-updater

Contact Information
-------------------

Owner
	Release Engineering, Fedora Infrastructure Team
Contact
	#fedora-apps, #fedora-releng, #fedora-admin, #fedora-noc
Servers
	pdc-web0{1,2}, pdc-backend01
Purpose
	Store metadata about composes and "component groups"

Description
-----------

The Product Definition Center (PDC) is a webapp and API designed for storing and
querying product metadata.  We automatically populate our instance with data
from our existing releng tools/processes.  It doesn't do much on its own, but
the goal is to enable us to develop more sane tooling down the road for future
releases.

The webapp is a django app running on pdc-web0{1,2}.  Unlike most of our other
apps, it does not use OpenID for authentication, but it instead uses SAML2.  It
uses `mod_auth_mellon` to achieve this (in cooperation with ipsilon).  The
webapp allows new data to be POST'd to it by admin users.

The backend is a `fedmsg-hub` process running on pdc-backend01.  It listens for
new composes over fedmsg and then POSTs data about those composes to PDC.  It
also listens for changes to the fedora atomic host git repo in pagure and
updates "component groups" in PDC to reflect what rpm components constitute
fedora atomic host.


For long-winded history and explanation, see the original Change document:
https://fedoraproject.org/wiki/Changes/ProductDefinitionCenter

Upgrading the Software
----------------------

There is an upgrade playbook in ``playbooks/manual/upgrade/pdc.yml`` which will
upgrade both the frontend and the backend if new packages are available.
Database schema upgrades should be handled automatically with a run of that
playbook.

Logs
----

Logs for the frontend are in `/var/log/httpd/error_log` on pdc-web0{1,2}.

Logs for the backend can be accessed with `journalctl -u fedmsg-hub -f` on
pdc-backend01.

Restarting Services
-------------------

The frontend runs under apache.  So either `apachectl graceful` or `systemctl
restart httpd` should do it.

The backend runs as a fedmsg-hub, so `systemctl restart fedmsg-hub` should
restart it.

Scripts
-------

The pdc-updater package (installed on pdc-backend01) provides three scripts:

- pdc-updater-audit
- pdc-updater-retry
- pdc-updater-initialize

A possible failure scenario is that we will lose a fedmsg message and the
backend will not update the frontend with info about that compose.  To detect
this, we provide the `pdc-updater-audit` command (which gets run once daily by
cron with emails sent to the releng-cron list).  It compare all of the entries
in PDC with all of the entries in kojipkgs and then raises an alert if there is
a discrepancy.

Another possible failure scenario is that the fedmsg message is published and
received correctly, but there is some processing error while handling it.  The
event occurred, but the import to the PDC db failed.  The `pdc-updater-audit`
script should detect this discrepancy, and then an admin will need to manually
repair the problem and retry the event with the `pdc-updater-retry` command.

If doomsday occurs and the whole thing is totally hosed, you can delete the db
and re-ingest all information available from releng with the
``pdc-updater-initialize`` tool.  (Creating the initial schema needs to happen
on pdc-web01 with the standard django settings.py commands.)

Manually Updating Information
-----------------------------

In general, you shouldn't have to do these things.  pdc-updater will
automatically create new releases and update information, but if you ever need
to manipulate PDC data, you can do it with the pdc-client tool.  A copy is
installed on pdc-backend01 and there are some credentials there you'll need, so
ssh there first.

Make sure that you are root so that you can read `/etc/pdc.d/fedora.json`.

Try listing all of the releases::

    $ pdc -s fedora release list

Deactivating an EOL release::

    $ pdc -s fedora release update fedora-21-updates --deactivate

.. note:: There are lots more attribute you can manipulate on a release (you can change
   the type, and rename them, etc..)  See `pdc --help` and `pdc release --help` for
   more information.

Listing all composes::

    $ pdc -s fedora compose list

We're not sure yet how to flag a compose as the Gold compose, but when we do,
the answer should appear here:
https://github.com/product-definition-center/product-definition-center/issues/428
