====================
Frameworks and Tools
====================

We attempt to use the same set of frameworks and tools across projects to
minimize the number of frameworks developers must keep in their heads.

Python
======

Flask
-----

`Flask`_ is a web microframework for Python based on `Werkzeug`_ and  `Jinja 2`_.
It is our preferred framework and all new applications should use it unless there
is a very good reason not to do so.

.. note::
    For historical reasons, you may find applications that don't use Flask. Other
    frameworks currently in use include `Django`_ and `Pyramid`_.

Flask is designed to be extensible, so it's common to use extensions with the
core flask library. A few common extensions are documented below.


Flask-SQLAlchemy
^^^^^^^^^^^^^^^^

`Flask-SQLAlchemy`_ integrates Flask with SQLAlchemy. It will configure a scoped
session for you, set up a declarative base class, and provide a convenient
:class:`flask_sqlalchemy.BaseQuery` sub-class for you.


SQLAlchemy
----------

`SQLAlchemy`_ is an SQL toolkit and Object Relational Mapper. It provides a
core set of tools (surprisingly called SQLAlchemy Core) for working with SQL
databases, as well as an Object Relational Mapper (SQLAlchemy ORM) which is
built using SQLAlchemy Core.

SQLAlchemy is quite flexible and provides a myriad of options. We use SQLAlchemy
with its `Declarative`_ extension to map SQL tables to Python classes. Once
mapped, instances of those Python classes are created from database rows using
the `Session`_ interfaces.


.. _Flask: http://flask.pocoo.org/
.. _Werkzeug: http://werkzeug.pocoo.org/
.. _Jinja 2: http://jinja.pocoo.org/
.. _Django: https://www.djangoproject.com/
.. _Pyramid: http://docs.pylonsproject.org/projects/pyramid/en/latest/
.. _SQLAlchemy: http://www.sqlalchemy.org/
.. _Declarative: http://docs.sqlalchemy.org/en/latest/orm/extensions/declarative/index.html
.. _Session: http://docs.sqlalchemy.org/en/latest/orm/session.html
.. _Flask-SQLAlchemy: http://flask-sqlalchemy.pocoo.org/
