.. title: Koschei SOP
.. slug: infra-koschei
.. date: 2016-09-29
.. taxonomy: Contributors/Infrastructure

===========
Koschei SOP
===========

Koschei is a continuous integration system for RPM packages.
Koschei runs package scratch builds after dependency change or
after time elapse and reports package buildability status to
interested parties.

Production instance: https://apps.fedoraproject.org/koschei
Staging instance:    https://apps.stg.fedoraproject.org/koschei

Contact Information
===================
Owner
	mizdebsk, msimacek
Contact
	#fedora-admin
Location
	Fedora Cloud
Purpose
	continuous integration system


Deployment
==========

Koschei deployment is managed by two Ansible playbooks::

  sudo rbac-playbook groups/koschei-backend.yml
  sudo rbac-playbook groups/koschei-web.yml

Description
===========
Koschei is deployed on two separate machines - ``koschei-backend`` and ``koschei-web``

Frontend (``koschei-web``) is a Flask WSGi application running with httpd.
It displays information to users and allows editing package groups and
changing priorities.

Backend (``koschei-backend``) consists of multiple services:

- ``koschei-watcher`` - listens to fedmsg events for complete builds and
  changes build states in the database

- ``koschei-repo-resolver`` - resolves package dependencies in given repo using
  hawkey and compares them with previous iteration to get a dependency diff. It
  resolves all packages in the newest repo available in Koji. The output is
  a base for scheduling new builds

- ``koschei-build-resolver`` - resolves complete builds in the repo in which they
  were done in Koji. Produces the dependency differences visible in the
  frontend

- ``koschei-scheduler`` - schedules new builds based on multiple criteria:

  - dependency priority - dependency changes since last build valued by their
    distance in the dependency graph
  - manual and static priorities - set manually in the frontend. Manual
    priority is reset after each build, static priority persists
  - time priority - time elapsed since the last build

- ``koschei-polling`` - polls the same types of events as koschei-watcher
  without reliance on fedmsg. Additionaly takes care of package list
  synchronization and other regularly executed tasks


Configuration
=============
Koschei configuration is in ``/etc/koschei/config-backend.cfg`` and
``/etc/koschei/config-frontend.cfg``, and is merged with the default
configuration in ``/usr/share/koschei/config.cfg`` (the ones in ``/etc``
overrides the defaults in ``/usr``). Note the merge is recursive. The
configuration contains all configurable items for all Koschei services
and the frontend. The alterations to configuration that aren't
temporary should be done through ansible playbook. Configuration
changes have no effect on already running services -- they need to be
restarted, which happens automatically when using the playbook.


Disk usage
==========
Koschei doesn't keep on disk anything that couldn't be recreated
easily - all important data is stored in PostgreSQL database,
configuration is managed by Ansible, code installed by RPM and so on.

To speed up operation and reduce load on external servers, Koschei
caches some data obtained from services it integrates with.  Most
notably, YUM repositories downloaded from Koji are kept in
``/var/cache/koschei/repodata``.  Each repository takes about 100 MB
of disk space.  Maximal number of repositories kept at time is
controlled by ``cache_l2_capacity`` parameter in
``config-backend.cfg`` (``config-backend.cfg.j2`` in Ansible).  If
repodata cache starts to consume too much disk space, that value can
be decreased - after restart, ``koschei-*-resolver`` will remove least
recently used cache entries to respect configured cache capacity.


Database
========
Koschei needs to connect to a PostgreSQL database, other database
systems are not supported. Database connection is specified in the
configuration under the ``database_config`` key that can contain the
following keys: ``username, password, host, port, database``.

After an update of koschei, the database needs to be migrated to new
schema. This happens automatically when using the upgrade playbook.
Alternatively, it can be executed manulally using::

  koschei-admin alembic upgrade head

The backend services need to be stopped during the migration.


Managing koschei services
=========================
Koschei services are systemd units managed through ``systemctl``. They can
be started and stopped independently in any order. The frontend is run
using httpd.


Suspespending koschei operation
===============================
For stopping builds from being scheduled, stopping the ``koschei-scheduler``
service is enough. For planned Koji outages, it's recommended to stop
``koschei-scheduler``. It is not necessary, as koschei can recover
from Koji errors and network errors automatically, but when Koji
builders are stopped, it may cause unexpected build failures that would
be reported to users. Other services can be left running as they
automatically restart themselves on Koji and network errors.


Limiting Koji usage
===================
Koschei is by default limited to 30 concurrently running builds. This
limit can be changed in the configuration under
``koji_config.max_builds`` key. There's also Koji load monitoring, that
prevents builds from being scheduled when Koji load is higher that
certain threshold. That should prevent scheduling builds during mass
rebuilds, so it's not necessary to stop scheduling during those.


Fedmsg notifications
====================
Koschei optionally supports sending fedmsg notifications for package
state changes. The fedmsg dispatch can be turned on and off in the
configuration (key ``fedmsg-publisher.enabled``). Koschei doesn't supply
configuration for fedmsg, it lets the library to load it's own (in
``/etc/fedmsg.d/``).


Setting admin announcement
==========================
Koschei can display announcement in web UI. This is mostly useful to
inform users about outages or other problems.

To set announcement, run as koschei user::

  koschei-admin set-notice "Koschei operation is currently suspended due to scheduled Koji outage"

or::

  koschei-admin set-notice "Sumbitting scratch builds by Koschei is currently disabled due to Fedora 23 mass rebuild"

To clear announcement, run as koschei user::

  koschei-admin clear-notice


Adding package groups
=====================
Packages can be added to one or more group.

To add new group named "mynewgroup", run as koschei user::

  koschei-admin add-group mynewgroup

To add new group named "mynewgroup" and populate it with some
packages, run as koschei user::

  koschei-admin add-group mynewgroup pkg1 pkg2 pkg3


Set package static priority
===========================
Some packages are more or less important and can have higher or lower
priority. Any user can change manual priority, which is reset after
package is rebuilt. Admins can additionally set static priority, which
is not affected by package rebuilds.

To set static priority of package "foo" to value "100", run as
koschei user::

  koschei-admin --collection f27 set-priority --static foo 100


Branching a new Fedora release
=====================
After branching occurs and Koji build targets have been created,
Koschei should be updated to reflect the new state. There is a special
admin command for this purpose, which takes care of copying the
configuration and also last builds from the history.

To branch the collection from Fedora 27 to Fedora 28, use the following::

  koschei-admin branch-collection f27 f28 -d 'Fedora 27' -t f28 --bugzilla-version 27

Then you can optionally verify that the collection configuration
is correct by visiting https://apps.fedoraproject.org/koschei/collections
and examining the configuration of the newly branched collection.
