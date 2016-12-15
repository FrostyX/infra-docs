.. title: Layered Image Build System
.. slug: layered-image-buildsys
.. date: 2016-12-15
.. taxonomy: Contributors/Infrastructure

==========================
Layered Image Build System
==========================

The `Fedora Layered Image Build System`_, often referred to as `OSBS`_
(OpenShift Build Service) as that is the upstream project that this is based on,
is used to build Layered Container Images in the Fedora Infrastructure via Koji.


Contents
========

1. Contact Information
2. Overview
3. Setup
4. Outage


Contact Information
===================

Owner
    Adam Miller (maxamillion)

Contact
    #fedora-admin, #fedora-releng, #fedora-noc, sysadmin-main, sysadmin-releng

Location
    osbs-control01, osbs-master01, osbs-node01, osbs-node02
    registry.fedoraproject.org, candidate-registry.fedoraproject.org

    osbs-control01.stg, osbs-master01.stg, osbs-node01.stg, osbs-node02.stg
    registry.stg.fedoraproject.org, candidate-registry.stg.fedoraproject.org

    x86_64 koji buildvms

Purpose
  Layered Container Image Builds


Overview
========

The build system is setup such that Fedora Layered Image maintainers will submit
a build to Koji via the ``fedpkg container-build`` command a ``docker``
namespace within `DistGit`_. This will trigger the build to be scheduled in
`OpenShift`_ via `osbs-client`_ tooling, this will create a custom
`OpenShift Build`_ which will use the pre-made buildroot `Docker`_ image that we
have created. The `Atomic Reactor`_ (``atomic-reactor``) utility will run within
the buildroot and prep the build container where the actual build action will
execute, it will also maintain uploading the `Content Generator`_ metadata back
to `Koji`_ and upload the built image to the candidate docker registry. This
will run on a host with iptables rules restricting access to the docker bridge,
this is how we will further limit the access of the buildroot to the outside
world verifying that all sources of information come from Fedora.

Completed layered image builds are hosted in a candidate docker registry which
is then used to pull the image and perform tests with `Taskotron`_. The
taskotron tests are triggered by a `fedmsg`_ message that is emitted from
`Koji`_ once the build is complete. Once the test is complete, taskotron will
send fedmsg which is then caught by the `RelEng Automation`_ Engine that will
run the Automatic Release tasks in order to push the layered image into a stable
docker registry in the production space for end users to consume.

For more information, please consult the `RelEng Architecture Document`_.


Setup
=====

The Layered Image Build System setup is currently as follows (more detailed view
available in the `RelEng Architecture Document`_):

::

    === Layered Image Build System Overview ===

         +--------------+                           +-----------+
         |              |                           |           |
         |   koji hub   +----+                      |  batcave  |
         |              |    |                      |           |
         +--------------+    |                      +----+------+
                             |                           |
                             V                           |
                 +----------------+                      V
                 |                |           +----------------+
                 |  koji builder  |           |                +-----------+
                 |                |           | osbs-control01 +--------+  |
                 +-+--------------+           |                +-----+  |  |
                   |                          +----------------+     |  |  |
                   |                                                 |  |  |
                   |                                                 |  |  |
                   |                                                 |  |  |
                   V                                                 |  |  |
        +----------------+                                           |  |  |
        |                |                                           |  |  |
        | osbs-master01  +------------------------------+           [ansible]
        |                +-------+                      |            |  |  |
        +----------------+       |                      |            |  |  |
             ^                   |                      |            |  |  |
             |                   |                      |            |  |  |
             |                   V                      V            |  |  |
             |        +-----------------+       +----------------+   |  |  |
             |        |                 |       |                |   |  |  |
             |        |  osbs-node01    |       |  osbs-node02   |   |  |  |
             |        |                 |       |                |   |  |  |
             |        +-----------------+       +----------------+   |  |  |
             |               ^                           ^           |  |  |
             |               |                           |           |  |  |
             |               |                           +-----------+  |  |
             |               |                                          |  |
             |               +------------------------------------------+  |
             |                                                             |
             +-------------------------------------------------------------+


Deployment
----------
The osbs-control01 host is where the `ansible-ansbile-openshift-ansible`_ role
is called from the `osbs-cluster.yml`_ playbook in order to configure the
OpenShift Cluster where OSBS is deployed on top of.


Operation
---------
Koji Hub will schedule the containerBuild on a koji builder via the
koji-containerbuild-hub plugin, the builder will then submit the build in
OpenShift via the koji-containerbuild-builder plugin which uses the osbs-client
python API that wraps the OpenShift API along with a custom OpenShift Build JSON
payload.

The Build is then scheduled in OpenShift and it's logs are captured by the koji
plugins. Inside the buildroot, atomic-reactor will upload the built container
image as well as provide the metadata to koji's content generator.


Outage
======

If Koji is down, then builds can't be scheduled but repairing Koji is outside
the scope of this document.

If either the candidate-registry.fedoraproject.org or registry.fedoraproject.org
Container Registries are unavailable, but repairing those is also outside the
scope of this document.

OSBS Failures
-------------

OpenShift Build System itself can have various types of failures that are known
about and the recovery procedures are listed below.

Ran out of disk space
~~~~~~~~~~~~~~~~~~~~~

Docker uses a lot of disk space, and while the osbs-nodes have been alloted what
is considered to be ample disk space for builds (since they are automatically
cleaned up periodically) it is possible this will run out.

To resolve this, run the following commands:

::

    # These command will clean up old/dead docker containers from old OpenShift
    # Pods

    $ for i in $(sudo docker ps -a | awk '/Exited/ { print $1 }'); do sudo docker rm $i; done

    $ for i in $(sudo docker images -q -f 'dangling=true'); do sudo docker rmi $i; done


    # This command should only be run on osbs-master01 (it won't work on the
    # nodes)
    #
    # This command will clean up old builds and related artifacts in OpenShift
    # that are older than 30 days (We can get more aggressive about this if
    # necessary, the main reason these still exist is in the event we need to
    # debug something. All build info we care about is stored in Koji.)

    $ oadm prune builds --orphans --keep-younger-than=720h0m0s --confirm

A node is broken, how to remove it from the cluster?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a node is having an issue, the following command will effectively remove it
from the cluster temporarily.

In this example, we are removing osbs-node01

::

    $ oadm manage-node osbs-node01.phx2.fedoraproject.org --schedulable=true


Container Builds are unable to access resources on the network
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes the Container Builds will fail and the logs will show that the
buildroot is unable to access networked resources (docker registry, dnf repos,
etc).

This is because of a bug in OpenShift v1.3.1 (current upstream release at the
time of this writing) where an OpenVSwitch flow is left behind when a Pod is
destroyed instead of the flow being deleted along with the Pod.

Method to confirm the issue is unfortunately multi-step since it's not
a cluster-wide issue but isolated to the node experiencing the problem.

First in the koji createContainer task there is a log file called
openshift-incremental.log and in there you will find a key:value in some JSON
output similar to the following:

::

    'openshift_build_selflink': u'/oapi/v1/namespaces/default/builds/cockpit-f24-6``


The last field of the value, in this example ``cockpit-f24-6`` is the OpenShift
build identifier. We need to ssh into ``osbs-master01`` and get information
about which node that ran on.

::

    # On osbs-master01
    #   Note: the output won't be pretty, but it gives you the info you need

    $ sudo oc get build cockpit-f25-3 -o yaml | grep osbs-node


Once you know what machine you need, ssh into it and run the following:

::

    $ sudo docker run --rm -ti buildroot /bin/bash'

    # now attempt to run a curl command

    $ curl https://google.com
    # This should get refused, but if this node is experiencing the networking
    # issue then this command will hang and eventually time out

How to fix:

Reboot the affected node that's experiencing the issue, when the node comes back
up OpenShift will rebuild the flow tables on OpenVSwitch and things will be back
to normal.

::

    systemctl reboot





.. CITATIONS/LINKS
.. _fedmsg: http://www.fedmsg.com/en/latest/
.. _Koji: https://fedoraproject.org/wiki/Koji
.. _Docker: https://github.com/docker/docker/
.. _OpenShift: https://www.openshift.org/
.. _Taskotron: https://taskotron.fedoraproject.org/
.. _docker-registry: https://docs.docker.com/registry/
.. _RelEng Automation: https://pagure.io/releng-automation
.. _osbs-client: https://github.com/projectatomic/osbs-client
.. _docker-distribution: https://github.com/docker/distribution/
.. _Atomic Reactor: https://github.com/projectatomic/atomic-reactor
.. _DistGit:
    https://fedoraproject.org/wiki/Infrastructure/VersionControl/dist-git
.. _OpenShift Build:
    https://docs.openshift.org/latest/dev_guide/builds.html
.. _Content Generator:
    https://fedoraproject.org/wiki/Koji/ContentGenerators
.. _RelEng Architecture Document:
    https://docs.pagure.org/releng/layered_image_build_service.html
.. _osbs-cluster:
    https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/playbooks/groups/osbs-cluster.yml
.. _ansible-ansible-openshift-ansible:
    https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/roles/ansible-ansible-openshift-ansible
