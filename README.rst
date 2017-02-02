Fedora Infrastructure Documentation
===================================
.. image:: https://readthedocs.org/projects/fedora-infra-docs/badge/?version=latest
        :alt: Documentation Status on Read the Docs
        :target: https://fedora-infra-docs.readthedocs.io/en/latest/

This repository documents standard operating procedures for applications Fedora
Infrastructure deploys and covers application development best practices for
those applications we maintain ourselves.

This is designed to be consumed as a Sphinx documentation project. Documentation
is available online on `pagure <https://docs.pagure.org/infra-docs/>`_ and on
`Read the Docs <https://fedora-infra-docs.readthedocs.io/>`_.

To build the documentation locally, create a virtualenv and install the requirements::

    $ sudo dnf install python-virtualenvwrapper
    $ mkvirtualenv -a $(pwd) infra-docs
    $ pip install -r requirements.txt

then build the documentation and open it in a browser of your choice::

    $ cd docs
    $ make html
    $ firefox _build/html/index.html

To leave the virtual environment::

    $ deactivate

Finally, when you want to work on the documentation, you can re-enter the
virtual environment with::

    $ workon infra-docs

It will automatically change the current working directory to the repository
root.
