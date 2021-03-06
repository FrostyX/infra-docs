.. title: Infrastructure virtio SIP
.. slug: infra-virtio
.. date: 2014-05-01
.. taxonomy: Contributors/Infrastructure

============
virtio notes
============

We have found that virtio is faster/more stable than emulating other cards
on our VMs.

To switch a VM to virtio:

- Remove from DNS if it's a proxy
- Log into the vm and shut it down
- Log into the virthost that the VM is on, and `sudo virsh edit <VM FQDN>`
- Add this line to the appropriate bridge interface(s)::

  <model type='virtio'/>

- Save/quit the editor
- `sudo virsh start <VM FQDN>`
- Re-add to DNS if it's a proxy
