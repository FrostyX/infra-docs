=========
Messaging
=========

Fedora uses many event-driven services triggered by messages. In the past, this
was done with ZeroMQ and `fedmsg`_. This has been replaced by an AMQP
message broker and the `fedora-messaging`_ for Python applications. This
documentation outlines the policies for sending and receiving messages. To
learn how to send and receive messages, see the fedora-messaging documentation.


Identity
========

In order to help with debugging, clients must configure the
`client_properties`_ option to include their application name under the ``app``
key. Clients should include the application version, if possible, in the
``app_version`` key.


Authentication
==============

When applications are deployed, clients must authenticate with the message
broker over a TLS connection using x509 certificates. An example AMQP URL for
this would be::

    amqp_url = "amqps://broker.example.com:5671/fedmsg?ssl_options=%7B%27keyfile%27%3A+%27%2Fetc%2Fpki%2Ffedora_messaging%2Fmykey.pem%27%2C+%27certfile%27%3A+%27%2Fetc%2Fpki%2Ffedora_messaging%2Fmycert.pem%27%7D

The ``ssl_options`` query parameter is a URL-encoded JSON object. You can create
it with::

    import urllib
    urllib.parse.urlencode({
        'ssl_options': {
            'certfile': '/etc/pki/fedora_messaging/mycert.pem',
            'keyfile': '/etc/pki/fedora_messaging/mykey.pem'
        }
    })


Authorization
=============

The message broker can use `virtual hosts`_ to allow multiple applications to
use the broker. The general purpose publish-subscribe virtual host is called
fedmsg and has its authorization policy is outlined below. If your application
is using a different virtual host for private messaging (for example, your
application uses Celery), different authorization rules apply.

Fedmsg Virtual Host
-------------------

AMQP clients using the fedmsg virtual host are restricted from creating many
resources. Additionally, topic authorization is enabled so only authorized
clients may publish to particular topics.

Resource Creation
^^^^^^^^^^^^^^^^^

In production, clients do not have permission to create exchanges, queues, or
bindings.  However, they can and should declare the exchanges, queues, and
bindings they expect to exist in their fedora-messaging configuration so that
if they do not exist, the application will fail with a helpful error message
about which resource is not available.

Exchanges, queues, bindings, and `virtual hosts`_ other objects are managed in
the broker using the Fedora Infrastructure Ansible project and must be declared
there. Consult the SOP for the Broker for details on how to do this.

Topics
^^^^^^

Clients must be `authorized to publish
<https://www.rabbitmq.com/access-control.html#topic-authorisation>`_ messages
with a given topic. The access control list is defined in the Fedora
Instrastructure Ansible project and you must ensure your client has been given
authorization to publish on its topic there.


.. _fedora-messaging: https://fedora-messaging.readthedocs.io/en/latest/
.. _client_properties: https://fedora-messaging.readthedocs.io/en/latest/configuration.html#client-properties
.. _virtual hosts: https://www.rabbitmq.com/vhosts.html
.. _fedmsg: https://fedmsg.readthedocs.io/en/stable/
