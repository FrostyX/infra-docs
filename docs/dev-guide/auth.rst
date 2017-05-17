==============
Authentication
==============

Fedora applications that require authentication should support, at a minimum,
authentication against `Ipsilon`_. Ipsilon is an Identity Provider that uses
a separate Identity Management system to perform authentication. In Fedora,
Ipsilon is currently backed by the `Fedora Account System`_. In the future, it
will be backed by `FreeIPA`_.

Ipsilon supports `OpenID 2.0`_, `OpenID Connect`_, `OAuth 2.0`_, and more.


Authentication
==============

All new applications should use OpenID Connect for user authentication.

.. note::
    Many existing applications use OpenID 2.0 and should eventually
    migrate to OpenID Connect.

OpenID Connect is an authentication layer built on top of OAuth 2.0 so
to understand OpenID Connect you should first be familiar with OAuth 2.0 and
its various flows prior to learning about OpenID Connect.

When requesting an access token in OAuth 2.0, clients are allowed to specify
the `scope`_ of the access token. This scope indicates what the token is allowed
to be used for. In most cases, your application should require a scope or scopes
of its own so users can issue access tokens that can only be used with a
particular application. To do so, consult the `Authentication Wiki page`_.


.. warning:: OpenID Connect `requires that the "openid" scope is requested
    <https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest>`_.
    Failing to do so will result in undefined behavior. In the case of Ipsilon,
    you won't have access to the UserInfo or recieve an ID token.


Libraries
=========

OAuthLib
--------

`OAuthLib`_ is a low-level implementation of OAuth 2.0 with OpenID Connect
support. It does not tie itself to a HTTP request framework. Typically, you
will only use this library indirectly. If you are investigating this library,
note that it is a library for both OAuth clients and OAuth providers. You
will be most interested in the `OAuth client`_ sub-package.


Requests-OAuthlib
-----------------

`Requests-OAuthlib`_ uses the `Requests`_ library with OAuthLib to provide an
easy-to-use interface for OAuth 2.0 clients. If you need to add support to an
application that doesn't have an extension for OAuthLib, you should use this
library.


Flask-OAuthlib
--------------

`Flask-OAuthlib`_ is a Flask extension that builds on top of Requests-OAuthlib.
It comes with plenty of examples in the `examples`_ directory of the repository.
Flask applications within Fedora Infrastructure should use this extension unless
there is a good reason not to (and that reason is documented here).


Pyramid-OAuthLib
----------------

`Pyramid-OAuthLib`_ is a Pyramid extension that uses OAuthlib. It does not appear
to be actively maintained, but it is a reasonable starting point for our few
Pyramid applications.


.. _Authentication Wiki page:
    https://fedoraproject.org/wiki/Infrastructure/Authentication
.. _examples: https://github.com/lepture/flask-oauthlib/tree/master/example
.. _Fedora Account System: https://admin.fedoraproject.org/accounts/
.. _Flask-OAuthlib: https://flask-oauthlib.readthedocs.io/en/latest/
.. _FreeIPA: http://www.freeipa.org/
.. _Ipsilon: https://ipsilon-project.org/
.. _OAuth 2.0: https://tools.ietf.org/html/rfc6749
.. _OAuth Client: https://oauthlib.readthedocs.io/en/latest/oauth2/clients/client.html
.. _OAuthLib: https://oauthlib.readthedocs.io/
.. _OpenID 2.0: https://openid.net/specs/openid-authentication-2_0.html
.. _OpenID Connect: https://openid.net/connect/
.. _oauth2client: https://oauth2client.readthedocs.io/
.. _Pyramid-OAuthLib: https://github.com/tilgovi/pyramid-oauthlib
.. _Requests-OAuthlib: https://requests-oauthlib.readthedocs.io/
.. _Requests: http://docs.python-requests.org/
.. _scope: https://tools.ietf.org/html/rfc6749#section-3.3
