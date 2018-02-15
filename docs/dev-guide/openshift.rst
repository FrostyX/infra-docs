=========
OpenShift
=========

OpenShift is a Kubernetes-based platform for running containers. The upstream project,
`OpenShift Origin`_, is what Red Hat bases the `OpenShift Container Platform`_ product
on. Fedora runs OpenShift Container Platform rather than OpenShift Origin.


Getting Started
===============

If you've never used OpenShift before a good place to start is with `MiniShift`_, which
deploys OpenShift Origin in a virtual machine.


OpenShift in Fedora Infrastructure
==================================

Fedora has two OpenShift deployments: `Staging OpenShift`_ and `Production OpenShift`_.
In addition to being the staging deployment of OpenShift itself, the staging deployment
is intended to be a place for developers to deploy the staging version of their applications.

Some features of OpenShift are not functional in Fedora's deployment, mainly due to the
lack of HTTP/2 support (at the time of this writing). Additionally, users are not allowed
to alter configuration, roll out new deployments, run builds, etc. in the web UI or CLI.


Web User Interface
------------------

Some of the web user interface is currently non-functional since it requires HTTP/2. The
rest is locked down to be read-only, making it of limited usefulness.


Command-line Interface
----------------------

Although the CLI is also locked down to be read only, it is possible to view logs and
request debugging containers, but only from batcave01. For example, to view the logs
of a deployment in staging::

    $ ssh batcave01.phx2.fedoraproject.org
    $ oc login os-master01.stg.phx2.fedoraproject.org
    You must obtain an API token by visiting https://os.stg.fedoraproject.org/oauth/token/request

    $ oc login os-master01.stg.phx2.fedoraproject.org --token=<Your token here>
    $ oc get pods
    librariesio2fedmsg-28-bfj52          1/1       Running     522        28d
    $ oc logs librariesio2fedmsg-28-bfj52


Deploying Your Application
--------------------------

Applications are deployed to OpenShift using `Ansible playbooks`_. You will need to
create an `Ansible Role`_ for your application. A role is made up of several YAML
files that define OpenShift `objects`_. To create these YAML objects you have two
options:

1. Copy and paste an existing role and do your best to rewrite all the files to work
   for your application. You will likely make mistakes which you won't find until you
   run the playbook and when you do learn that your configuration is invalid, it won't
   be clear where you messed up.

2. Set up your own deployment of OpenShift where you can click through the web UI to
   create your application (and occasionally use the built-in text editor when the UI
   doesn't have buttons for a feature you need). Once you've done that, you can export
   all the configuration files and drop them into the infra ansible repository. They
   will be "messy" with lots of additional data OpenShift adds for you (including
   old revisions of the configuration).

Both approaches have their downsides. #1 has a very long feedback cycle as you edit the
file, commit it to the infra repository, and then run the playbook. #2 generates most of
the configuration, but will produce crufty files. Additionally, you will likely not have
your OpenShift deployment set up the same way Fedora does so you still may produce
configurations that won't work.

You will likely need (at a minimum) the following objects:

* A `BuildConfig`_ - This defines how your container is built.
* An `ImageStream`_ - This references a "stream" of container images and lets you trigger
  deployments or image builds based on changes in a stream.
* A `DeploymentConfig`_ - This defines how your container is deployed (how many replicas,
  what ports are available, etc)
* A `Service`_ - An internal load balancer that routes traffic to your pods.
* A `Route`_ - This exposes a Service as a host name.


.. _Ansible Role: https://infrastructure.fedoraproject.org/infra/ansible/roles/openshift-apps/
.. _Ansible Playbooks: https://infrastructure.fedoraproject.org/infra/ansible/playbooks/openshift-apps/
.. _Staging OpenShift: https://os.stg.fedoraproject.org/
.. _Production OpenShift: https://os.fedoraproject.org/
.. _OpenShift Origin: https://www.openshift.org/
.. _OpenShift Container Platform: https://www.openshift.com/
.. _MiniShift: https://www.openshift.org/minishift/
.. _objects: https://docs.openshift.com/container-platform/latest/architecture/core_concepts/index.html
.. _BuildConfig: https://docs.openshift.com/container-platform/latest/architecture/core_concepts/builds_and_image_streams.html#builds
.. _ImageStream: https://docs.openshift.com/container-platform/latest/architecture/core_concepts/builds_and_image_streams.html#image-streams
.. _DeploymentConfig: https://docs.openshift.com/container-platform/latest/architecture/core_concepts/deployments.html
.. _Service: https://docs.openshift.com/container-platform/latest/architecture/core_concepts/pods_and_services.html#services
.. _Route: https://docs.openshift.com/container-platform/latest/architecture/networking/routes.html
