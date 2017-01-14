.. title: Ansible Infrastructure SOP 
.. slug: infra-ansible
.. date: 2015-03-03
.. taxonomy: Contributors/Infrastructure

=======================================
Ansible infrastructure SOP/Information. 
=======================================

Background
==========

Fedora infrastructure used to use func and puppet for system change management. 
We are now using ansible for all system change mangement and ad-hoc tasks.

Overview
========

Ansible runs from batcave01 or backup01. These hosts run a ssh-agent that 
has unlocked the ansible root ssh private key. (This is unlocked manually 
by a human with the passphrase each reboot, the passphrase itself is not 
stored anywhere on the machines). Using 'sudo -i' sysadmin-main members 
can use this agent to access any machines with the ansible root ssh public
key setup, either with 'ansible' for one-off commands or 'ansible-playbook'
to run playbooks.  

Playbooks are idempotent (or should be). Meaning you should be able to re-run 
the same playbook over and over and it should get to a state where 0 items
are changing.

Additionally (see below) there is a rbac wrapper that allows members of some
other groups to run playbooks against specific hosts. 

git repo(s)
-----------

There are 2 git repositories associated with ansible: 

/git/ansible on batcave01. 
	This is a public repository. Never commit private data to this repo. 
	You can access it also via a cgit web interface at: 
	https://infrastructure.fedoraproject.org/cgit/ansible.git/
	You can check it out on batcave01 with: 'git clone /git/ansible'
	You can also use it remotely if you have your ssh set to proxy your access
	via bastion01: ``git clone ssh://batcave01/git/ansible``

	Users in the 'sysadmin' group have commit access to this repo. 
	All commits are emailed to 'sysadmin-members' as well as announced
	on IRC in #fedora-noc. 

/git/ansible-private on batcave01.
	This is a private repository for passwords and other sensitive data. 
	It is not available in cgit, nor should it be cloned or copied remotely. 
	It's only available to members of 'sysadmin-main'. 

Cron job/scheduled runs
-----------------------

With use of run_ansible-playbook_cron.py that is run daily via cron we walk through
playbooks and run them with `--check --diff` params to perform a dry-run.

This way we make sure all the playbooks are idempotent and there is no
unexpected changes on servers (or playbooks).

Logging
-------

We have in place a callback plugin that stores history for any ansible-playbook runs 
and then sends a report each day to sysadmin-logs-members with any CHANGED or FAILED
actions. Additionally, there's a fedmsg plugin that reports start and end of ansible
playbook runs to the fedmsg bus. Ansible also logs to syslog verbose reporting of when
and what commands and playbooks were run. 

role based access control for playbooks
---------------------------------------

There's a wrapper script on batcave01 called 'rbac-playbook' that allows non sysadmin-main
members to run specific playbooks against specific groups of hosts. This is part of the
ansible_utils package. The upstream for ansible_utils is: https://bitbucket.org/tflink/ansible_utils

To add a new group:

1. add the playbook name and sysadmin group to the rbac-playbook (ansible-private repo)
2. add that sysadmin group to sudoers on batcave01 (also in ansible-private repo)

To use the wrapper::

sudo rbac-playbook playbook.yml

Directory setup
================

Inventory
---------

The inventory directory tells ansible all the hosts that are managed by it and 
the groups they are in. All files in this dir are concatenated together, so you 
can split out groups/hosts into separate files for readability. They are in ini 
file format. 

Additionally under the inventory directory are host_vars and group_vars subdirectories. 
These are files named for the host or group and containing variables to set 
for that host or group. You should strive to set variables in the highest level 
possible, and precedence is in: global, group, host order. 

Vars
----

This directory contains global variables as well as OS specific variables. Note that 
in order to use the OS specific ones you must have 'gather_facts' as 'True' or ansible
will not have the facts it needs to determine the OS. 

Roles
-----

Roles are a collection of tasks/files/templates that can be used on any host or group
of hosts that all share that role. In other words, roles should be used except in cases
where configuration only applies to a single host. Roles can be reused between hosts and
groups and are more portable/flexable than tasks or specific plays. 

Scripts
-------

In the ansible git repo under scripts are a number of utilty scripts for sysadmins. 

Playbooks
---------

In the ansible git repo there's a directory for playbooks. The top level contains 
utility playbooks for sysadmins. These playbooks perform one-off functions or gather 
information. Under this directory are hosts and groups playbooks. These playbooks are 
for specific hosts and groups of hosts, from provision to fully configured. You should
only use a host playbook in cases where there will never be more than one of that thing. 

Tasks
-----

This directory contains one-off tasks that are used in playbooks. Some of these should
be migrated to roles (we had this setup before roles existed in ansible). Those that 
are truely only used on one host/group could stay as isolated tasks. 

Syntax
------

Ansible now warns about depreciated syntax. Please fix any cases you see related to 
depreciation warnings.

Templates use the jinja2 syntax. 

Libvirt virtuals
================
* TODO: add steps to make new libvirt virtuals in staging and production
* TODO: merge in new-hosts.txt

Cloud Instances
===============
* TODO: add how to make new cloud instances
* TODO: merge in from ansible README file. 

rdiff-backups
=============
see: https://infrastructure.fedoraproject.org/infra/docs/rdiff-backup.rst

Additional Reading/Resources
============================

Upstream docs: 
  https://docs.ansible.com/

Example repo with all kinds of examples:
  * https://github.com/ansible/ansible-examples
  * https://gist.github.com/marktheunissen/2979474

Jinja2 docs:
  http://jinja.pocoo.org/docs/
