.. title: fedmsg Certificates SOP
.. slug: infra-fedmsg-certs
.. date: 2013-04-08
.. taxonomy: Contributors/Infrastructure

===================================================
fedmsg (Fedora Messaging) Certs, Keys, and CA - SOP
===================================================

X509 certs, private RSA keys, Certificate Authority, and Certificate
Revocation List.

Contact Information
===================

Owner
  Messaging SIG, Fedora Infrastructure Team
Contact
  #fedora-admin, #fedora-apps, #fedora-noc
Servers
  - app0[1-7]
  - packages0[1-2]
  - fas0[1-3]
  - pkgs01 
  - busgateway01,
  - value0{1,3}
  - releng0{1,4}
  - relepel03
Purpose
	      Certify fedmsg messages come from authentic sources.

Description
===========

fedmsg sends JSON-encoded messages from many services to a zeromq messaging
bus.  We're not concerned with encrypting the messages, only with signing them
so an attacker cannot spoof.

Every instance of each service on each host has its own cert and private key,
signed by the CA.  By convention, we name the certs <service>-<fqdn>.{crt,key}
For instance, bodhi has the following certs:

- bodhi-app01.phx2.fedoraproject.org
- bodhi-app02.phx2.fedoraproject.org
- bodhi-app03.phx2.fedoraproject.org
- bodhi-app01.stg.phx2.fedoraproject.org
- bodhi-app02.stg.phx2.fedoraproject.org
- more

Scripts to generate new keys, sign them, and revoke them live in the ansible
repo in ``ansible/roles/fedmsg/files/cert-tools/``.  The keys and certs
themselves (including ca.crt and the CRL) live in the private repo in
``private/fedmsg-certs/keys/``

fedmsg is locally configured to find the key it needs by looking in
``/etc/fedmsg.d/ssl.py`` which is kept in ansible in
``ansible/roles/fedmsg/templates/fedmsg.d/ssl.py.erb``.

Each service-host has its own key.  This means:

- A key is not shared across multiple instances of a service on
  different machines.  i.e., bodhi on app01 and bodhi on app02 should have
  different key/cert pairs.
 
- A key is not shared across multiple services on a host.  i.e., mediawiki
  on app01 and bodhi on app01 should have different key/cert pairs.

The attempt here is to minimize the number of potential attack vectors.
Each private key should be readable only by the service that needs it.
bodhi runs under mod_wsgi in apache and should run as its own unique bodhi
user (not as apache).  The permissions for its.phx2.fedoraproject.org
private_key, when deployed by ansible, should be read-only for that local
bodhi user.

For more information on how fedmsg uses these certs see
http://fedmsg.readthedocs.org/en/latest/crypto.html


Configuring the Scripts
=======================

Usage of the main scripts is described in more detail below.  They are
located in ``ansible/rolesfedmsg/files/cert-tools``.

Before you use them, you'll need to point them at the right directory to
modify.  By default, this is ``~/private/fedmsg-certs/keys/``.  You
can change that by editing ``ansible/roles/fedmsg/files/cert-tools/vars`` in
the event that you have the private repo checked out to an alternate location.

There are other configuration values defined in that script.  Most will not
need to be changed.

Wiping and Rebuilding Everything
================================

There is a script in ``ansible/roles/fedmsg/files/cert-tools/`` named
``rebuild-all-fedmsg-certs``.  You can run it with no arguments to wipe out
the old and generate a new CA root certificate, a signing cert and key, and
all key/cert pairs for all service-hosts.

.. note:: Warning -- Obviously, this will wipe everything.  Do you want that?

Adding a new key for a new service-host
=======================================

First, checkout the ansible private repo as that's where the keys are going
to be stored.  The scripts will assume this is checked out to ~/private.

In ``ansible/roles/fedmsg/files/cert-tools`` run::

  $ source ./vars
  $ ./build-and-sign-key <service>-<fqdn>

For instance, if we bring up a new app host, app10.phx2.fedoraproject.org,
we'll need to generate a new cert/key pair for each fedmsg-enabled service
that will be running on it, so you'd run::

  $ source ./vars
  $ ./build-and-sign-key shell-app10.phx2.fedoraproject.org
  $ ./build-and-sign-key bodhi-app10.phx2.fedoraproject.org
  $ ./build-and-sign-key mediawiki-app10.phx2.fedoraproject.org

Just creating the keys isn't quite enough, there are four more things you'll
need to do.

The private keys are created in your checkout of the private repo under
~/private/private/fedmsg-certs/keys . There will be four files for each cert
you created: <hexdigits>.pem (ex: 5B.pem) and <service>-<fqdn>.{crt,csr,key}
git add, commit, and push all of those.

Second, You need to edit
``ansible/roles/fedmsg/files/cert-tools/rebuild-all-fedmsg-certs``
and add the argument of the commands you just ran, so that next time certs need
to be blown away and recreated, the new service-hosts will be included.
For the examples above, you would need to add to the list::

  shell-app10.phx2.fedoraproject.org
  bodhi-app10.phx2.fedoraproject.org
  mediawiki-app10.phx2.fedoraproject.org

You need to ensure that the keys are distributed to the host with the proper
permissions.  Only the bodhi user should be able to access bodhi's private
key.  This can be accomplished by using the ``fedmsg::certificate`` in
ansible.  It should distribute your new keys to the correct hosts and
correctly permission them.

Lastly, if you haven't already updated the global fedmsg config, you'll need
to.  You need to add your new service-node to ``fedmsg.d/endpoint.py`` and
to ``fedmsg.d/ssl.py``.  Those can be found in
``ansible/roles/fedmsg/templates/fedmsg.d``.  See
http://fedmsg.readthedocs.org/en/latest/config.html for more information on
the layout and meaning of those files.

Revoking a key
==============

In ``ansible/roles/fedmsg/files/cert-tools`` run::

  $ source ./vars
  $ ./revoke-full <service>-<fqdn>

This will alter ``private/fedmsg-certs/keys/crl.pem`` which should be
picked up and served publicly, and then consumed by all fedmsg consumers
globally.

``crl.pem`` is publicly available at http://fedoraproject.org/fedmsg/crl.pem

.. note:: Even though crl.pem lives in the private repo, we're just keeping
          it there for convenience.  It really *should* be served publicly,
          so don't panic.  :)

.. note:: At the time of this writing, the CRL is not actually used.  I need
          one publicly available first so we can test it out.
