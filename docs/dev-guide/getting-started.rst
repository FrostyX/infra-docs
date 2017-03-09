
.. _dev-getting-started:

===============
Getting Started
===============

This document is intended to guide you through your first contribution to a
Fedora Infrastructure project. It assumes you are already familiar with the
`git`_ version control system and the `Python`_ programming language.


Development Environment
=======================

The Fedora Infrastructure team uses `Ansible`_ and `Vagrant`_ to set up
development environments for the majority of our projects. It's recommended
that you develop on a Fedora host, but that is not strictly required.

To install `Ansible`_ and `Vagrant`_ on Fedora, run::

    $ sudo dnf install vagrant libvirt vagrant-libvirt vagrant-sshfs ansible

Projects will provide a ``Vagrantfile.example`` file in the root of their
repository if they support using `Vagrant`_. Copy this to ``Vagrantfile``,
adjust it as you see fit, and then run::

    $ vagrant up
    $ vagrant reload
    $ vagrant ssh

This will create a new virtual machine, configure it with `Ansible`_, restart
it to ensure you're running the latest updates, and then SSH into the virtual
machine.

Individual projects will provide detailed instructions for their particular
setup.


Finding a Project
=================

Fedora Infrastructure applications are either on `GitHub`_ in the
`fedora-infra`_ organization, or on `Pagure`_. Check out the issues tagged with
`easyfix`_ for an issue to fix.


.. _git: https://git-scm.com/
.. _Python: https://www.python.org/
.. _Ansible: https://www.ansible.com/
.. _Vagrant: https://vagrantup.com/
.. _GitHub: https://github.com/
.. _fedora-infra: https://github.com/fedora-infra
.. _Pagure: https://pagure.io/
.. _easyfix: https://fedoraproject.org/easyfix/
