.. title: Infrastructure Unbound SOP 
.. slug: infra-unbound
.. date: 2013-11-22
.. taxonomy: Contributors/Infrastructure

==========================
Fedora Infra Unbound Notes
==========================

Sometimes, especially after updates/reboots you will see alerts like this::

  18:46:55 < zodbot> PROBLEM - unbound-tummy01.fedoraproject.org/Unbound 443/tcp is WARNING: DNS WARNING - 0.037 seconds response time (dig returned an error status) (noc01)
  18:51:06 < zodbot> PROBLEM - unbound-tummy01.fedoraproject.org/Unbound 80/tcp is WARNING: DNS WARNING - 0.035 seconds response time (dig returned an error status) (noc01)

To correct this, restart unbound on the relevant node (in the example case
above, unbound-tummy01), by running the restart_unbound Ansible playbook from
lockbox01.::

  sudo -i ansible-playbook /srv/web/infra/ansible/playbooks/restart_unbound.yml --extra-vars="target=unbound-tummy01.fedoraproject.org"
