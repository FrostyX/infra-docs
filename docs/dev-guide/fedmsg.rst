======
fedmsg
======

`fedmsg`_ is a ZeroMQ-based messaging library used throughout Fedora
Infrastructure applications. It uses a publish/subscribe design so applications
can decide what messages they're interested in receiving.

.. warning:: fedmsg does not guarantee message delivery. Messages will be lost and
    your application should never depend on the reliable delivery of fedmsgs to
    function.

Topics
======

Existing Topics
---------------

There are many existing `topics`_ in Fedora Infrastructure.


New Topics
----------

When creating new message topics, please use the following format::

 org.fedoraproject.ENV.CATEGORY.OBJECT[.SUBOBJECT].EVENT

Where:

 - ``ENV`` is one of `dev`, `stg`, or `production`.
 - ``CATEGORY`` is the name of the service emitting the message -- something
   like `koji`, `bodhi`, or `fedoratagger`
 - ``OBJECT`` is something like `package`, `user`, or `tag`
 - ``SUBOBJECT`` is something like `owner` or `build` (in the case where
   ``OBJECT`` is `package`, for instance)
 - ``EVENT`` is a verb like `update`, `create`, or `complete`.

All 'fields' in a topic **should**:

 - Be `singular` (Use `package`, not `packages`)
 - Use existing fields as much as possible (since `complete` is already used
   by other topics, use that instead of using `finished`).

**Furthermore**, the *body* of messages will contain the following envelope:

- A ``topic`` field indicating the topic of the message.
- A ``timestamp`` indicating the seconds since the epoch when the message was
  published.
- A ``msg_id`` bearing a unique value distinguishing the message.  It is
  typically of the form <YEAR>-<UUID>.  These can be used to uniquely query for
  messages in the datagrepper web services.
- A ``crypto`` field indicating if the message is signed with the ``X509``
  method or the ``gpg`` method.
- An ``i`` field indicating the sequence of the message if it comes from a
  permanent service.
- A ``username`` field indicating the username of the process that published
  the message (sometimes, ``apache`` or ``fedmsg`` or something else).
- Lastly, the application-specific body of the message will be contained in a
  nested ``msg`` dictionary.


.. _fedmsg: https://fedmsg.readthedocs.io/
.. _topics: https://fedora-fedmsg.readthedocs.org/
