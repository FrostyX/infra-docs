.. title: rdiff-backup Infrastructure SOP
.. slug: infra-rdiff
.. date: 2013-11-01
.. taxonomy: Contributors/Infrastructure

================
rdiff-backup SOP
================

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	 Phoenix
Servers
	 backup03 and others
Purpose
	 backups of critical data

Description
===========

We are now running a rdiff-backup of all our critical data on a daily basis. 
This allows us to keep incremental changes over time as well has have a recent copy
in case of disaster recovery. 

The backups are run from backup03 every day at 22:10UTC as root. 
All config is in ansible. 

The cron job checks out the ansible repo from git, then runs ansible-playbook with
the rdiff-backup playbook. This playbook looks at variables to decide which 
machines and partitions to backup. 

- First, machines in the backup_clients group in inventory are operated on. 
  If a host is not in that group it is not backed up via rdiff-backup. 

- Next, any machines in the backup_clients group will have their /etc and /home 
  directories backed up by the server running rdiff-backup and using the rdiff-backup
  ssh key to access the client. 

- Next, if any of the hosts in backup_clients have a variable set for 
  host_backup_targets, those directories will also be backed up in the same 
  manner as above with the rdiff-backup ssh key. 

For each backup an email will be sent to sysadin-backup-members with a summary. 

Backups are stored on a netapp volume, so in addition to the incrementals 
that rdiff-backup provides there are netapp snapshots. This netapp volume is 
mounted on /fedora_backups and is running dedup on the netapp side. 

Rebooting backup03
==================

When backup03 is rebooted, you must restart the ssh-agent and reload the 
rdiff-backup ssh key into that agent so backups can take place. 

::

  sudo -i
  ssh-agent -s > sshagent
  source sshgent
  ssh-add .ssh/rdiff-backup-key

Adding a new host to backups
============================

1. add the host to the backup_clients inventory group in ansible. 

2. If you wish to backup more than /etc and /home, add a variable to: 
    inventory/host_vars/fqdn like: 
    host_backup_targets: ['/srv']

3. On the client to be backed up, install rdiff-backup. 

4. On the client to be backed up, install the rdiff-backup ssh public key to 
    ``/root/.ssh/authorized_keys``
    It should be restricted from::

     from="10.5.126.161,192.168.1.64" 
     
    and command can be restricted to::

      command="rdiff-backup --server --restrict-update-only"

Restoring from backups
======================
rdiff backup keeps a copy of the most recent version of files on disk, so if you 
wish to restore the last backup copy, simply rsync from backup03. If you wish an older
incremental, see rdiff-backup man page for how to specify the exact time. 

Retention 
=========

Backups are currently kept forever, but likely down the road we will look at 
pruning them some to match available space. 

Public_key:
===========

::
  
  ssh-dss
  AAAAB3NzaC1kc3MAAACBAJr3xqn/hHIXeth+NuXPu9P91FG9jozF3Q1JaGmg6szo770rrmhiSsxso/Ibm2mObqQLCyfm/qSOQRynv6tL3tQVHA6EEx0PNacnBcOV7UowR5kd4AYv82K1vQhof3YTxOMmNIOrdy6deDqIf4sLz1TDHvEDwjrxtFf8ugyZWNbTAAAAFQCS5puRZF4gpNbaWxe6gLzm3rBeewAAAIBcEd6pRatE2Qc/dW0YwwudTEaOCUnHmtYs2PHKbOPds0+Woe1aWH38NiE+CmklcUpyRsGEf3O0l5vm3VrVlnfuHpgt/a/pbzxm0U6DGm2AebtqEmaCX3CIuYzKhG5wmXqJ/z+Hc5MDj2mn2TchHqsk1O8VZM+1Ml6zX3Hl4vvBsQAAAIALDt5NFv6GLuid8eik/nn8NORd9FJPDBJxgVqHNIm08RMC6aI++fqwkBhVPFKBra5utrMKQmnKs/sOWycLYTqqcSMPdWSkdWYjBCSJ/QNpyN4laCmPWLgb3I+2zORgR0EjeV2e/46geS0MWLmeEsFwztpSj4Tv4e18L8Dsp2uB2Q==
  root@backup03-rdiff-backup
