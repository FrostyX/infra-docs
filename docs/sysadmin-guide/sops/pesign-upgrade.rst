.. title: Pesign Upgrades and Reboots
.. slug: infra-pesign-maintenance 
.. date: 2013-05-29
.. taxonomy: Contributors/Infrastructure

=======================
Pesign upgrades/reboots
=======================

Fedora has (currently) 2 special builders. These builders are used to 
build a small set of packages that need to be signed for secure boot. 
These packages include: grub2, shim, kernel, pesign-test-app

When rebooting or upgrading pesign on these machines, you have to 
follow a special process to unlock the signing keys. 

Contact Information
===================

Owner
  Fedora Release Engineering, Kernel/grub2/shim/pesign maintainers
Contact
  #fedora-admin, #fedora-kernel
Servers
  bkernel01, bkernel02
Purpose
  Upgrade or restart singning keys on kernel/grub2/shim builders

Procedure
===========

0. Coordinate with pesign maintainers or pesign-test-app commiters as well
as releng folks that have the pin to unlock the signing key. 

1. remove builder from koji::

    koji disable-host bkernel01.phx2.fedoraproject.org

2. Make sure all builds have completed. 

3. Stop existing processes::

    service pcscd stop
    service pesign stop

4. Perform updates or reboots. 

5. Restart services (if you didn't reboot)::

    service pcscd start
    service pesign start

6. Unlock signing key::

    pesign-client -t "OpenSC Card (Fedora Signer)" -u
    (enter pin when prompted)

7. Make sure no builds are in progress, then Re-add builder to koji, remove other builder::

    koji enable-host bkernel01.phx2.fedoraproject.org
    koji disable-host bkernel02.phx2.fedoraproject.org

8. Have a commiter send a build of pesign-test-app and make sure it's signed correctly. 

9. If so, repeat process with second builder. 

