.. title: Guest Disk Resize SOP
.. slug: infra-guest-disk-resize
.. date: 2012-06-13
.. taxonomy: Contributors/Infrastructure

=====================
Guest Disk Resize SOP
=====================

Resize disks in our kvm guests

Contents
========

1. Contact Information
2. How to do it

  1. KVM/libvirt Guests

Contact Information
===================

Owner: 
  Fedora Infrastructure Team
Contact: 
  #fedora-admin, sysadmin-main
Location: 
  PHX, Tummy, ibiblio, Telia, OSUOSL
Servers: 
  All xen servers, kvm/libvirt servers.
Purpose: 
  Resize guest disks

How to do it
============

KVM/libvirt Guests
------------------

1. SSH to the kvm server and resize the guest's logical volume. If you
    want to be extra careful, make a snapshot of the LV first::

      lvcreate -n [guest name]-snap -L 10G -s /dev/VolGroup00/[guest name] 
    
    Optional, but always good to be careful

2. Shutdown the guest::

    sudo virsh shutdown [guest name]

3. Disable the guests lv::

    lvchange -an /dev/VolGroup00/[guest name]

4. Resize the lv::

    lvresize -L [NEW TOTAL SIZE]G /dev/VolGroup00/[guest name]

    or

    lvresize -L +XG /dev/VolGroup00/[guest name]
    (to add X GB to the disk)

5. Enable the lv::

    lvchange -ay /dev/VolGroup00/[guest name]

6. Bring the guest back up::

    sudo virsh start [guest name]

7. Login into the guest::

    sudo virsh console [guest name]
    You may wish to boot single user mode to avoid services coming up and going down again

8. On the guest, run::

    fdisk /dev/vda

9. Delete the the LVM partition on the guest you want to add space to and
   recreate it with the maximum size. Make sure to set its type to LV (8e)
 
10. Run partprobe::

      partprobe

11. Check the size of the partition::

      fdisk -l /dev/vdaN

    If this still reflects the old size, then reboot the guest and verify
    that its size changed correctly when it comes up again.

12. Login to the guest again, and run::

      pvresize /dev/vdaN

13. A vgs should now show the new size. Use lvresize to resize the root lv::

      lvresize -L [new root partition size]G /dev/GuestVolGroup00/root

      (pvs will tell you how much space is available)

14. Finally, resize the root partition::

      resize2fs /dev/GuestVolGroup00/root
      (If the root fs is ext4)

      or

      xfs_growfs /dev/GuestVolGroup00/root
      (if the root fs is xfs)

    verify that everything worked out, and delete the snapshot you made
    if you made one.
