.. title: The New Hotness SOP
.. slug: hotness-sop
.. date: 2017-01-31
.. taxonomy: Contributors/Infrastructure

.. _hotness-sop:

===============
The New Hotness
===============
`the-new-hotness <https://github.com/fedora-infra/the-new-hotness/>`_ is a
`fedmsg consumer <http://www.fedmsg.com/en/latest/consuming/#the-hub-consumer-approach>`_
that subscribes to `release-monitoring.org <https://release-monitoring.org/>`_ fedmsg
notifications to determine when a package in Fedora should be updated. For more details
on the-new-hotness, consult the `project documentation <http://the-new-hotness.readthedocs.io/>`_.


Contact Information
===================
Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
    phx2.fedoraproject.org

Servers

- hotness01.phx2.fedoraproject.org
- hotness01.stg.phx2.fedoraproject.org

Purpose
	 File issues when upstream projects release new versions of a package


Deploying a New Version
=======================
As of January 31, 2017, the-new-hotness is not packaged for Fedora or EPEL. When upstream
tags a new version in Git and you are building a new version (from the specfile in the upstream
repository), you will need to build it into the :ref:`infra-repo`.

1. Build the SRPM with ``koji build epel7-infra the-new-hotness-<version>-<release>.src.rpm``. If
   you do not have permission to perform this build (it fails with permission denied), ask for help
   in #fedora-admin.

2. Consult the upstream changelog. If necessary, adjust the Ansible configuration for
   the-new-hotness.

3. Update the host. At the moment this is done with shell access to the host and running::

   $ sudo -i yum clean all
   $ sudo -i yum update the-new-hotness

4. Ensure the configuration is up-to-date by running this on batcave01::

   $ sudo rbac-playbook -l staging groups/hotness.yml  # remove the "-l staging" to update prod

All done!


Monitoring Activity
===================
It can be nice to check up on the-new-hotness to make sure its behaving correctly.
You can see all the Bugzilla activity using the
`user activity query <https://bugzilla.redhat.com/page.cgi?id=user_activity.html>`_ (staging uses
`partner-bugzilla.redhat.com <https://partner-bugzilla.redhat.com/page.cgi?id=user_activity.html>`_)
and querying for the ``upstream-release-monitoring@fedoraproject.org`` user.

You can also view all the Koji tasks dispatched by the-new-hotness. For example, you can see the
`failed tasks <https://koji.fedoraproject.org/koji/tasks?state=failed&owner=hotness>`_
it has created.
