.. title: Loopabull
.. slug: loopabull
.. date: 2017-01-17
.. taxonomy: Contributors/Infrastructure


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
    Pierre-Yves Chibon (pingou)

Contact
    #fedora-admin, #fedora-releng, #fedora-noc, sysadmin-main, sysadmin-releng

Location
    loopabull01.phx2.fedoraproject.org
    loopabull01.stg.phx2.fedoraproject.org

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

Loopabull is deployed on two hosts, one for the production instance:
``loopabull01.prod.phx2.fedoraproject.org`` and one for the staging instance:
``loopabull01.stg.phx2.fedoraproject.org``.

Each host is running loopabull with 5 workers reacting to fedmsg
notifications.

Expanding loopabull
===================

The documentation to expand loopabull's usage is documented at:
`https://pagure.io/Fedora-Infra/loopabull-tasks <https://pagure.io/Fedora-Infra/loopabull-tasks>`_


Outage
======

In the event that loopabull isn't responding or isn't running playbooks as it
should be, the following scenarios should be approached.

What is going on?
-----------------

There are a few commands that may help figuring out what is going:

* Check the status of the different services:

::

    systemctl |grep loopabull

* Follow the logs of the different services:

::

    journalctl -lfu loopabull -u loopabull@1 -u loopabull@2 -u loopabull@3 \
        -u loopabull@4 -u loopabull@5

If a playbook returns a non-zero error code, the worker running it will be
stopped. If that happens, you may want to carefully review the logs to
assess what lead to this situation so it can be prevented in the future.


Network Interruption
--------------------

Sometimes if the network is interrupted, the loopabull service will hang because
the fedmsg listener will hold a dead socket open. The service and its workers
simply needs to be restarted at that point.

::

    systemctl restart loopabull loopabull@1 loopabull@2 loopabull@3 \
        loopabull@4 loopabull@5


.. CITATIONS/LINKS
.. _Ansible: https://www.ansible.com/
.. _fedmsg: http://www.fedmsg.com/en/latest/
.. _loopabull: https://github.com/maxamillion/loopabull
.. _playbook: http://docs.ansible.com/ansible/playbooks.html
.. _Release Engineering Automation: https://pagure.io/releng-automation
.. _releng docs: https://docs.pagure.org/releng/automation_engine.html
.. _extra variables:
    https://github.com/ansible/ansible/blob/devel/docs/man/man1/ansible-playbook.1.asciidoc.in
