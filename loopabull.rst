.. title: Loopabull
.. slug: loopabull
.. date: 2017-01-17
.. taxonomy: Contributors/Infrastructure


.. ##########################################################################
.. NOTE: This document is currently under construction. The service described
         herein is not yet in production.
.. ##########################################################################


=========
Loopabull
=========

`Loopabull`_ is an event-driven `Ansible`_-based automation engine. This is used
for various tasks, originally slated for `Release Engineering Automation`_.

Contents
========

1. Contact Information
2. Overview
3. Setup
4. Outage


Contact Information
===================

Owner
    Adam Miller (maxamillion)

Contact
    #fedora-admin, #fedora-releng, #fedora-noc, sysadmin-main, sysadmin-releng

Location

    TBD

Purpose
    Event Driven Automation of tasks within the Fedora Infrastructure and Fedora
    Release Engineering


Overview
========

The `loopabull`_ system is setup such that an event will take place within the
infrastructure and a `fedmsg`_ is sent, then loopabull will consume that
message, trigger an `Ansible`_ `playbook`_ that shares a name with the fedmsg
topic, and provide the payload of the fedmsg to the playbook as `extra
variables`_.


Setup
=====

The setup is relatively simple, the Overview above describes it and a more
detailed version can be found in the `releng docs`.

::

    +-----------------+             +-------------------------------+
    |                 |             |                               |
    |    fedmsg       +------------>|  Looper                       |
    |                 |             |   (fedmsg handler plugin)     |
    |                 |             |                               |
    +-----------------+             +-------------------------------+
                                                    |
                                                    |
                 +-------------------+              |
                 |                   |              |
                 |                   |              |
                 |    Loopabull      +<-------------+
                 |  (Event Loop)     |
                 |                   |
                 +---------+---------+
                           |
                           |
                           |
                           |
                           V
                +----------+-----------+
                |                      |
                |   ansible-playbook   |
                |                      |
                +----------------------+

Deployment
----------

TBD


Outage
======

In the event that loopabull isn't responding or isn't running playbooks as it
should be, the following scenarios should be approached.

Network Interruption
--------------------

Sometimes if the network is interrupted, the loopabull service will hang because
the fedmsg listener will hold a dead socket open. The service simply needs to be
restarted at that point.

::

    systemctl restart loopabull.service

.. CITATIONS/LINKS
.. _Ansible: https://www.ansible.com/
.. _fedmsg: http://www.fedmsg.com/en/latest/
.. _loopabull: https://github.com/maxamillion/loopabull
.. _playbook: http://docs.ansible.com/ansible/playbooks.html
.. _Release Engineering Automation: https://pagure.io/releng-automation
.. _releng docs: https://docs.pagure.org/releng/automation_engine.html
.. _extra variables:
    https://github.com/ansible/ansible/blob/devel/docs/man/man1/ansible-playbook.1.asciidoc.in
