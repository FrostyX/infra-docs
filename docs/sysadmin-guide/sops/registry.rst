.. title: Container registry SOP
.. slug: infra-registry
.. date: 2017-05-16
.. taxonomy: Contributors/Infrastructure

======================
Container registry SOP
======================

Fedora uses the `Docker Distribution <https://github.com/docker/distribution>`_ container registry
to host its container images.

Production instance: https://registry.fedoraproject.org
CDN instance: https://cdn.registry.fedoraproject.org


Contact information
===================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin
Persons
 bowlofeggs
 maxamillion
Location
 Phoenix
Servers
 docker-registry01
Purpose
 Serve Fedora's container images


Configuring all nodes
=====================

Run this command from the `ansible` checkout to configure all nodes in production::

        sudo -i ansible-playbook $(pwd)/playbooks/groups/docker-registry.yml


Upgrades
========

Fedora infrastructure uses the registry packaged and distributed in the ``rhel7-extras`` repository.
Thus, there is no special upgrade procedure - a simple ``yum update`` will do.


System architecture
===================

The container registry is hosted in a fairly simple design. There are two hosts that run Docker
Distribution to serve the registry API, and these hosts are behind a load balancer. These hosts will
respond to all requests except for requests for blobs. Requests for blobs will receive a 302
redirect to https://cdn.registry.fedoraproject.org, which is a caching proxy hosted by CDN 77. The
primary goal of serving the registry API ourselves is so that we can serve the container manifests
over TLS so that users can be assured they are receiving the correct image blobs when they retrieve
them. We do not rely on signatures since we do not have a Notary instance.

The two registry instances are configured not to cache their data, and use Gluster to replicate
their shared storage. This way, changes to one registry should appear in the other quickly.


Troubleshooting
===============

Logs
----

You can monitor the registry via the systemd journal::

        sudo journalctl -f -u docker-distribution
