=====
Tests
=====

Tests make development easier for both veteran project contributors and
newcomers alike. Most projects use the `unittest`_ framework for tests
so you should familiarize yourself with this framework.

.. note::
    Writing tests can be a great way to get involved with a project. It's an
    opportunity to get familiar with the codebase and the code submission and
    review process. Check the project's code coverage and write a test for a
    piece of code missing coverage!

Patches should be accompanied by one or more tests to demonstrate the feature
or bugfix works. This makes the review process much easier since it allows the
reviewer to run your code with very little effort, and it lets developers know
when they break your code.


Test Organization
=================

Having a standard test layout makes it easy to find tests. When adding new
tests, follow the following guidelines:

1. Each module in the application should have a corresponding test module.
   These modules should be organized in the test package to mirror the package
   they test. That is, if the package contains the ``<package>/server/push.py``
   module, the test module should be in a module called
   ``<test_root>/server/test_push.py``.

2. Within each test module, follow the `unittest code organization guidelines`_.

3. Include documentation blocks for each test case that explain the goal of the
   test.

4. Avoid using mock unless absolutely necessary. It's easy to write tests using
   mock that only assert that mock works as expected. When testing code that
   makes HTTP requests, consider using `vcrpy <https://pypi.python.org/pypi/vcrpy>`_.

.. note::
    You may find projects that do not follow this test layout. In those cases,
    consider re-organizing the tests to follow the layout described here and
    follow the established conventions for that project until that happens.


Test Runners
============

Projects should include a way to run the tests with ease locally and the steps to run
the tests should be documented. This should be the same way the continuous integration
(Jenkins, TravisCI, etc.) tool runs the tests.

There are many test runners available that can discover `unittest`_ based tests.
These include:

* `unittest`_ itself via ``python -m unittest discover``

* `pytest`_

* `nose2`_

Projects should choose whichever runner best suits them.

.. note::
    You may find projects using the `nose`_ test runner. nose is in maintenance
    mode and, according to their documentation, will likely cease without a new
    maintainer. They recommend using `unittest`_, `pytest`_, or `nose2`_.

.. _tox-config:

Tox
===

`Tox`_ is an easy way to run your project's tests (using a Python test runner)
using multiple Python interpreters. It also allows you to define arbitrary test
environments, so it's an excellent place to run the code style tests and to
ensure the project's documentation builds without errors or warnings.

Here's an example ``tox.ini`` file that runs a project's unit tests in Python
2.7, Python 3.4, Python 3.5, and Python 3.6. It also runs `flake8`_ on the
entire codebase and builds the documentation with the "warnings treated as
errors" Sphinx flag enabled::

    [tox]
    envlist = py27,py34,py35,py36,lint,docs
    # If the user is missing an interpreter, don't fail
    skip_missing_interpreters = True

    [testenv]
    deps =
        -rtest-requirements.txt
    # Substitute your test runner of choice
    commands =
        py.test

    [testenv:docs]
    changedir = docs
    deps =
        sphinx
        sphinxcontrib-httpdomain
        -rrequirements.txt
    whitelist_externals =
        mkdir
        sphinx-build
    commands=
        mkdir -p _static
        sphinx-build -W -b html -d {envtmpdir}/doctrees .  _build/html

    [testenv:lint]
    deps =
        flake8 > 3.0
    commands =
        python -m flake8 {posargs}

    [flake8]
    show-source = True
    max-line-length = 100
    exclude = .git,.tox,dist,*egg


Coverage
========

`coverage`_ is a good way to collect test coverage statistics. `pytest`_ has a
`pytest-cov`_ plugin that integrates with `coverage`_ and `nose-cov`_ provides
integration for the `nose`_ test runner.

It's possible (and recommended) to have the test suite fail if the coverage
percentage goes down. This example ``.coveragerc``::

    [run]
    # Track what conditional branches are covered.
    branch = True
    include =
        my_python_package/*

    [report]
    # Fail if the coverage is not 100%
    fail_under = 100
    # Display results with up 1/100th of a percent accuracy.
    precision = 2
    exclude_lines =
        pragma: no cover

        # Don't complain if tests don't hit defensive assertion code
        raise AssertionError
        raise NotImplementedError

        if __name__ == .__main__.:
    omit =
        my_python_package/tests/*

causes coverage (and any test running plugins using coverage) to fail if the
coverage level is not 100%. New projects should enforce 100% test coverage.
Existing projects should ensure test coverage does not drop to accept a pull
request and should increase the minimum test coverage until it is 100%.

.. note::
    `coverage`_ has great `exclusion`_ support, so you can exclude individual
    lines, conditional branches, functions, classes, and whole source files
    from your coverage report. If you have code that doesn't make sense to
    have tests for, you can exclude it from your coverage report. Remember
    to leave a comment explaining why it's excluded!

.. _coverage: https://pypi.python.org/pypi/coverage/
.. _exclusion: https://coverage.readthedocs.io/en/coverage-4.3.4/excluding.html
.. _flake8: https://pypi.python.org/pypi/flake8
.. _pytest: http://docs.pytest.org/en/latest/contents.html
.. _pytest-cov: https://pypi.python.org/pypi/pytest-cov
.. _nose: https://nose.readthedocs.io/en/latest/
.. _nose2: http://nose2.readthedocs.io/en/latest/
.. _nose-cov: https://pypi.python.org/pypi/nose-cov
.. _unittest: https://docs.python.org/3.6/library/unittest.html
.. _unittest code organization guidelines:
    https://docs.python.org/3.6/library/unittest.html#organizing-test-code
.. _tox: https://pypi.python.org/pypi/tox
