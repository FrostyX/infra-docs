
.. _sysadmin-guide:

==========================
System Administrator Guide
==========================

Welcome to the Fedora Infrastructure system administration guide.


.. _sysadmin-getting-started:

Getting Started
===============

If you haven't already, you should complete the general :ref:`getting-started`
guide. Once you've completed that, you're ready to get involved in the
`Fedora Infrastructure Apprentice`_ group.


Fedora Infrastructure Apprentice
--------------------------------

The `Fedora Infrastructure Apprentice`_ group in the Fedora Account System
grants read-only access to many Fedora infrastructure machines. This group is
used for new folks to look around at the infrastructure setup, check machines
and processes and see where they might like to contribute moving forward. This
also allows apprentices to examine and gather info on problems, then propose
solutions.

.. note::
    This group will be pruned often of inactive folks who miss the monthly
    email check-in on the `infrastructure mailing list`_. There's nothing
    personal in this and you're welcome to re-join later when you have more
    time, we just want to make sure the group only has active members.

Members of the `Fedora Infrastructure Apprentice`_ group have ssh/shell access
to many machines, but no sudo rights or ability to commit to the `Ansible
repository`_ (but they do have read-only access). Apprentice can, however,
contribute to the infrastructure documentation by making a pull request to the
`infra-docs`_ repository. Access is via the bastion.fedoraproject.org machine
and from there to each machine. See the :ref:`ssh-sop` for instructions on how
to set up SSH. You can see a list of hosts that allow apprentice access by
using::

   $  ansible/scripts/hosts_with_var_set -o fas_client_groups=fi-apprentice'

from a checkout of the `Ansible repository`_. The Ansible repository is hosted
on batcave01.phx2.fedoraproject.org in ``/git/ansible``.


Selecting a Ticket
------------------

Start by checking out the `easyfix tickets`_. Tickets marked with this tag are
a good place for apprentices to learn how things are setup, and also contribute
a fix.

Since apprentices do not have commit access to the `Ansible repository`_, you
should make your change, produce a patch with ``git diff``, and attach it to
the infrastructure ticket you are working on. It will then be reviewed.


.. _sops:

Standard Operating Procedures
=============================

Below is a table of contents containing all the standard operating procedures
for Fedora Infrastructure applications. For information on how to write a new
standard operating procedure, consult the guide on :ref:`develop-sops`.

.. toctree::
   :maxdepth: 2
   :caption: Standard Operating Procedures:

   sops/2-factor
   sops/accountdeletion
   sops/anitya
   sops/ansible
   sops/apps-fp-o
   sops/archive-old-fedora
   sops/arm
   sops/askbot
   sops/badges
   sops/basset
   sops/bastion-hosts-info
   sops/bladecenter
   sops/blockerbugs
   sops/bodhi
   sops/bugzilla2fedmsg
   sops/bugzilla
   sops/cloud
   sops/collectd
   sops/contenthosting
   sops/copr
   sops/cyclades
   sops/darkserver
   sops/database
   sops/datanommer
   sops/denyhosts
   sops/departing-admin
   sops/dns
   sops/docs.fedoraproject.org
   sops/fas-notes
   sops/fas-openid
   sops/fedmsg-certs
   sops/fedmsg-gateway
   sops/fedmsg-introduction
   sops/fedmsg-irc
   sops/fedmsg-new-message-type
   sops/fedmsg-relay
   sops/fedmsg-websocket
   sops/fedocal
   sops/fedorahosted-fedmsg
   sops/fedorahosted-project-cleanup
   sops/fedorahostedrename
   sops/fedorahosted-repo-setup
   sops/fedorahosted
   sops/fedorapackages
   sops/fedorapastebin
   sops/fedora-releases
   sops/fedorawebsites
   sops/fmn
   sops/freemedia
   sops/freenode-irc-channel
   sops/gather-easyfix
   sops/geoip-city-wsgi
   sops/github2fedmsg
   sops/github
   sops/gitweb
   sops/guestdisk
   sops/guestedit
   sops/guestmigrate
   sops/haproxy
   sops/hosted_git_to_svn
   sops/hotfix
   sops/hotness
   sops/ibm-drive-replacement
   sops/ibm_rsa_ii
   sops/infra-git-repo
   sops/infra-hostrename
   sops/infra-raidmismatch
   sops/infra-repo
   sops/infra-retiremachine
   sops/infra-yubikey
   sops/ipsilon
   sops/iscsi
   sops/jenkins-fedmsg
   sops/kerneltest-harness
   sops/kickstarts
   sops/koji-builder-setup
   sops/koji
   sops/koschei
   sops/layered-image-buildsys
   sops/linktracking
   sops/loopabull
   sops/mailman
   sops/making-ssl-certificates
   sops/massupgrade
   sops/mastermirror
   sops/memcached
   sops/mirrorhiding
   sops/mirrormanager
   sops/mirrormanager-S3-EC2-netblocks
   sops/mote
   sops/nagios
   sops/netapp
   sops/new-hosts
   sops/nonhumanaccounts
   sops/nuancier
   sops/openvpn
   sops/orientation
   sops/outage
   sops/packagedatabase
   sops/packagereview
   sops/pdc
   sops/pesign-upgrade
   sops/planetsubgroup
   sops/privatefedorahosted
   sops/publictest-dev-stg-production
   sops/rdiff-backup
   sops/requestforresources
   sops/resultsdb
   sops/reviewboard
   sops/scmadmin
   sops/selinux
   sops/sigul-upgrade
   sops/sshaccess
   sops/sshknownhosts
   sops/staging-infra
   sops/staging
   sops/stagingservers
   sops/status-fedora
   sops/syslog
   sops/taskotron
   sops/torrentrelease
   sops/unbound
   sops/virt-image
   sops/virtio
   sops/virt-notes
   sops/voting
   sops/wiki
   sops/zodbot


.. _Fedora Infrastructure Apprentice:
    https://admin.fedoraproject.org/accounts/group/view/fi-apprentice
.. _Ansible repository:
    https://infrastructure.fedoraproject.org/infra/ansible/
.. _infrastructure mailing list:
    https://lists.fedoraproject.org/admin/lists/infrastructure.lists.fedoraproject.org/
.. _easyfix tickets:
    https://pagure.io/fedora-infrastructure/issues?status=Open&tags=easyfix
.. _infra-docs: https://pagure.io/infra-docs/
