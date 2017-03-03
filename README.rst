===================================
Fedora Infrastructure Documentation
===================================
.. image:: https://readthedocs.org/projects/fedora-infra-docs/badge/?version=latest
        :alt: Documentation Status on Read the Docs
        :target: https://fedora-infra-docs.readthedocs.io/en/latest/

This repository documents standard operating procedures for applications Fedora
Infrastructure deploys and covers application development best practices for
those applications we maintain ourselves.

This is designed to be consumed as a Sphinx documentation project. Documentation
is available online on `Pagure`_ and on `Read the Docs`_.


Contributing
============

This project is maintained on `our Pagure repository`_. Contributions are
reviewed and accepted by submitting a `pull request`_ to our Pagure repository.
If you're not familiar with the pull request workflow, check out the Pagure
`documentation on pull requests`_.

Documentation is written in reStructedText. See Sphinx's `reStructuredText
Primer`_ for a brief introduction to the concepts and format.


Local Builds
============

To build the documentation locally, create a virtualenv and install the requirements::

    $ sudo dnf install python-virtualenvwrapper
    $ mkvirtualenv -a $(pwd) infra-docs
    $ pip install -r requirements.txt

Then build the documentation and open it in a browser of your choice::

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


.. _Pagure: https://docs.pagure.org/infra-docs/
.. _Read the Docs: https://fedora-infra-docs.readthedocs.io/
.. _our Pagure repository: https://pagure.io/infra-docs
.. _pull request: https://pagure.io/infra-docs/pull-requests
.. _documentation on pull requests:
   https://docs.pagure.org/pagure/usage/pull_requests.html
.. _reStructuredText Primer:
    http://www.sphinx-doc.org/en/stable/rest.html
