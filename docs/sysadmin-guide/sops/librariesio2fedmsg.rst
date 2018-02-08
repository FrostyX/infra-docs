.. title: librariesio2fedmsg SOP
.. slug: infra-librariesio2fedmsg
.. date: 2018-01-18
.. taxonomy: Contributors/Infrastructure

======================
librariesio2fedmsg SOP
======================

librariesio2fedmsg is a small service that converts Server-Sent Events from
`libraries.io`_ to fedmsgs.

librariesio2fedmsg is an instance of `sse2fedmsg`_ using the `libraries.io
firehose`_ running on `OpenShift`_ and publishes its fedmsgs through the
busgateway01.phx2.fedoraproject.org relay using the
``org.fedoraproject.prod.sse2fedmsg.librariesio`` topic.


Updating
========

sse2fedmsg is installed directly from its git repository, so once a new release
is tagged in sse2fedmsg, just update the tag in the git URL provided to pip in
the `build config`_.


Deploying
=========

Run the playbook to apply the new OpenShift configuration::

  $ sudo rbac-playbook openshift-apps/librariesio2fedmsg.yml


.. _libraries.io: https://libraries.io/
.. _libraries.io firehose: http://firehose.libraries.io/events
.. _sse2fedmsg: https://github.com/fedora-infra/sse2fedmsg
.. _OpenShift: https://os.fedoraproject.org/
.. _build config: https://infrastructure.fedoraproject.org/infra/ansible/roles/openshift-apps/librariesio2fedmsg/files/
