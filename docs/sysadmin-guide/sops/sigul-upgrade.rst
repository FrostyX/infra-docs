.. title: Sigul Servers Maintenance SOP
.. slug: infra-sigul-mainenance
.. date: 2015-02-04
.. taxonomy: Contributors/Infrastructure

==============================
Sigul servers upgrades/reboots
==============================

Fedora currently has 1 sign-bridge and 2 sign-vault machines for primary, there
is a similar setup for secondary architectures. When upgrading or rebooting
these machines, some special steps must be taken to ensure everything is
working as expected.

Contact Information
===================

Owner
	Fedora Release Engineering
Contact
	#fedora-admin, #fedora-noc
Servers
	sign-vault03, sign-vault04, sign-bridge02, secondary-bridge01.qa
Purpose
	Upgrade or restart sign servers

Description
===========
0. Coordinate with releng on timing. Make sure no signing is happening, and
none is planned for a bit. 

Sign-bridge02, secondary-bridge01.qa:

  1. Apply updates or changes

  2. Reboot virtual instance

  3. Once it comes back, start the sigul_bridge service and enter empty password.

Sign-vault03/04: 

  1. Determine which server is currently primary. It's the one that has the
      floating ip address for sign-vault02 on it. 

  2. Login to the non primary server via serial or management console. 
      (There is no ssh access to these servers)

  3. Take a lvm snapshot::

        lvcreate --size 5G --snapshot --name YYYMMDD /dev/mapper/vg_signvault04-lv_root

      Replace YYMMDD with todays year, month, day and the vg with the correct name 
      Then apply updates. 

  4. Confirm the server comes back up ok, login to serial console or management
      console and start the sigul_server process. Enter password when prompted. 

  5. On the primary server, down the floating ip address::

        ip addr del 10.5.125.75 dev eth0

  6. On the secondary server, up the floating ip address::

        ip addr add 10.5.125.75 dev eth0

  7. Have rel-eng folks sign some packages to confirm all is working. 

  8. Update/reboot the old primary server and confirm it comes back up ok. 

.. note:: Changes to database

    When making any changes to the database (new keys, etc), it's important to 
    sync the data from the primary to the secondary server. This process is
    currently manual. 
