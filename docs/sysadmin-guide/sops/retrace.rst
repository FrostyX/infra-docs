.. title: Retrace SOP
.. slug: infra-retrace
.. date: 2017-15-17
.. taxonomy: Contributors/Infrastructure

===========
retrace SOP
===========

  Retrace server - provides complete tracebacks for unhandled crashes and
  show aggregated information for developers.

Contact Information
-------------------

Owner:
  Fedora QA Devel, Fedora Infrastructure Team, ABRT team
Contact:
  #abrt, #fedora-admin, #fedora-noc
Servers:
  retrace*, faf*

Purpose:
  Provides complete tracebacks for unhandled crashes and
  show aggregated information for developers.

Description
-----------

The physical server (retrace.fedoraproject.org) runs two main services:
retrace-server and FAF.


Retrace-server
==============

The upstream for retrace server lives at:

  https://github.com/abrt/retrace-server

When a user has the ABRT client installed and a process crashes with
an unhandled exception (e.g., traceback or core dump), the user can send
a request to retrace-server. The server will install the same set of packages
plus debuginfo, and will return a traceback to the user that includes function
names instead of plain pointers. This information is useful for debugging.

The upstream retrace-server allows users to upload coredumps through a web
interface, but the Fedora instance disables this feature.

FAF
===

When a user decides to report a crash, data is sent to FAF. ABRT can also be
configured to send microreports automatically, if desired.

FAF can aggregate similar reports into one entity (called a Problem). FAF
provides a nice web interface for developers, allowing them to see crashes of
their packages. It lives at:

  https://retrace.fedoraproject.org/faf/

Playbook
--------

The playbook is split into several roles. There are two main roles

 * abrt/faf
 * abrt/retrace

These roles are copied from upstream. You should never update it directly.
The new version can be fetched from upstream using:

 # cd ansible/abrt
 # rm -rf faf retrace
 # ansible-galaxy install -f -r requirements.yml --ignore-errors -p ./

You should review the new differences, and commit and push.

Then there are some roles which are local for our instance:

 * abrt/faf-local - This is run *before* abrt/faf.
 * abrt/retrace-local - This is run *after* abrt/retrace.
 * abrt/retrace-local-pre - This is run *before* abrt/retrace.

Services
--------

FAF and retrace-server are web applications; only httpd is required.

Cron
----

FAF and retrace-server each have cron tasks. They are *not* installed under
/etc/cron* but are installed as user cron jobs for the 'faf' and 'retrace'
users.

You can list those crons using:

 * sudo -u faf crontab -l
 * sudo -u retrace crontab -l

All cronjobs should be Ansible managed. Just make sure if you delete some
cron from Ansible that it does not remain on the server (not always possible
with state=absent).

Directories
-----------

- /srv/ssd - fast disk, used for PostgreSQL storage
- /srv - big fat disk, used for storing packages. Mainly:
  - /srv/faf/lob
  - /srv/retrace
- /srv/faf/db-backup/ - Daily backups of DB. No rotating yet. Needs to be
    manually deleted occasionally.
- /srv/faf/lob/InvalidUReport/ - Invalid reports, can be pretty big.
    No automatic removal too. Need to be purged manually occasionally.

Front-page
----------

The main web page is handled by the `abrt-server-info-page` package, which can be
controlled using:

 /usr/lib/python2.7/site-packages/abrt-server-info-page/config.py

DB
--

Only FAF uses a database. We use our own instance of PostgreSQL. You can
connect to it using:

  sudo -u faf psql faf
