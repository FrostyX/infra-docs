==========
Code Style
==========

We attempt to maintain a consistent coding style across projects so contributors
do not have to keep dozens of different styles in their head as they move from
project to project.


Python
======

We follow the `PEP8 <https://www.python.org/dev/peps/pep-0008/>`_ style guide
for Python. Projects should make it easy to check new code for PEP8 violations
preferably by `flake8 <https://pypi.python.org/pypi/flake8>`_. It is up to the
individual project to choose an enforcement method, but it should be clearly
documented and continuous integration tests should ensure code is correctly
styled before merging pull requests.

.. note::
    There are a few PEP8 rules which will vary from project to project. For example,
    the maximum line length might vary. The test suite should enforce this.


Enforcement
-----------
Projects should automatically enforce code style. How a project does so is
up to the maintainers, but several good options are documented here.

Tox
^^^

:ref:`tox-config` is an excellent way to test style. ``flake8`` looks in,
among other places, ``tox.ini`` for its configuration so if you're already
using tox this is a good place to place your configuration. For example, adding
the following snippet to your ``tox.ini`` file will run ``flake8`` as part of
your test suite::

  [testenv:lint]
  deps =
      flake8 > 3.0
  commands =
      python -m flake8 {posargs}

  [flake8]
  show-source = True
  max-line-length = 100
  exclude = .git,.tox,dist,*egg

Unit Test
^^^^^^^^^

If you're not using ``tox``, you can add a unit test to your test suite::

    """This module runs flake8 on a subset of the code base"""
    import os
    import subprocess
    import unittest

    # Adjust this as necessary to ensure REPO_PATH resolves to the root of your
    # repository.
    REPO_PATH = os.path.abspath(os.path.dirname(os.path.join(os.path.dirname(__file__), '../')))

    class TestStyle(unittest.TestCase):
        """Run flake8 on the repository directory as part of the unit tests."""

        def test_code_with_flake8(self):
            """Assert the code is PEP8-compliant"""

            # enforced_paths = [
            #     'mypackage/pep8subpackage/',
            #     'mypackage/a_module.py',
            # ]
            # enforced_paths = [os.path.join(REPO_PATH, p) for p in enforced_paths]

            # If your entire codebase is not already PEP8 compliant, you can enforce
            # the style incrementally using ``enforced_paths``.
            #
            # flake8_command = ['flake8', '--max-line-length', '100'] + enforced_paths
            flake8_command = ['flake8', '--max-line-length', '100', REPO_PATH]

            self.assertEqual(subprocess.call(flake8_command), 0)


    if __name__ == '__main__':
        unittest.main()
