
Contribution Guidelines
=======================
This documents some basic guidelines all Fedora Infrastructure projects should
follow. It also serves as a template for individual project's contribution
guidelines. You should feel free to copy this file into a project's
documentation and adjust it as necessary.


Code Style
----------
We follow the `PEP8 <https://www.python.org/dev/peps/pep-0008/>`_ style guide for Python.
You can add a unit test to your test suite to automatically enforce this style with
`flake8 <https://pypi.python.org/pypi/flake8>`_::

    """This module runs flake8 on a subset of the code base"""
    import os
    import subprocess
    import unittest

    # Adjust this as necessary to ensure REPO_PATH resolves to the root of your
    # repository.
    REPO_PATH = os.path.abspath(
        os.path.dirname(os.path.join(os.path.dirname(__file__), '../')))


    class TestStyle(unittest.TestCase):
        """Run flake8 on the repository directory as part of the unit tests."""

        def test_code_with_flake8(self):
            """Assert the code is PEP8-compliant"""

            enforced_paths = [
                'pagure/proxy.py',
                'pagure/pfmarkdown.py',
            ]
            enforced_paths = [os.path.join(REPO_PATH, p) for p in enforced_paths]

            # If your entire codebase is already PEP8 compliant, you can remove the
            # ``enforced_paths`` and replace it with ``REPO_PATH``.
            flake8_command = ['flake8', '--max-line-length', '100'] + enforced_paths
            self.assertEqual(subprocess.call(flake8_command), 0)


    if __name__ == '__main__':
        unittest.main()

This way, as long as the tests pass, you can be sure the pull request follows the
style guide!


Testing
-------
Tests make development easier for both veteran project contributors and
newcomers alike. Setting up a pleasant testing environment for the start saves
countless headaches later. New projects should enforce 100% test coverage.
Unless there are special project requirements, the project should use the
`unittest <https://docs.python.org/3.6/library/unittest.html>`_ test framework
for tests.

Try to make each test function as minimal as possible, as this makes diagnosing
regressions much easier. The sample Flask project in this repository contains a
test suite that is a good example of test organization. It closely follows the
suggested code organization in `unittest <https://docs.python.org/3.6/library/unittest.html>`_.

Patches should be accompanied by one or more tests to demonstrate the feature
or bugfix works. This makes the review process much easier for developers since
it allows them to run your code with very little effort.


Documentation
-------------
Projects should contain a `Sphinx <http://www.sphinx-doc.org/>`_ documentation
project that documents how to use the application, how to deploy and maintain
the application, and how to interface with the application. The best way to
maintain accurate documentation is to use the
`autodoc <http://www.sphinx-doc.org/en/stable/tutorial.html#autodoc>`_ feature
of Sphinx. This turns Python documentation blocks into HTML pages. This way, most
of your documentation can be written in the code itself.

Python Modules
^^^^^^^^^^^^^^
All modules should contain a documentation block at the top of the file that describes
the module's purpose and documents module-level attributes it provides as part
of its public interface. In the case of a package's ``__init__.py``, this should
document the package's purpose. It is very helpful to make liberal use of Sphinx's
`cross-referencing <http://www.sphinx-doc.org/en/stable/domains.html#cross-referencing-python-objects>`_
of Python objects for easy navigation in the HTML documentation.

HTTP APIs
^^^^^^^^^
Many projects provide an HTTP-based API.
`sphinxcontrib-httpdomain <http://pythonhosted.org/sphinxcontrib-httpdomain/>`_
is a Sphinx extension used to automatically generate REST API documentation.
If the project uses the Flask framework, it is possible to provide this
extension with the Flask application object and have it discover all the
routes automatically. It's helpful to add the
``.. :quickref: [<resource>;] <short description>`` rst comment in all route
function doc strings. This allows endpoints to be grouped by resource.

Style
^^^^^
The style of the documentation blocks is left up to the individual project.
By default, Sphinx expects ReStructuredText. However, it has several extensions
that allow alternate styles. These styles are referred to the
`Google style <http://www.sphinx-doc.org/en/1.5.2/ext/example_google.html>`_
and the `NumPy style <http://www.sphinx-doc.org/en/1.5.2/ext/example_numpy.html>`_.
Pick a style, document your choice, and stick with it throughout your project.


Development Environment
-----------------------
Vagrantfiles and Dockerfiles are good.
