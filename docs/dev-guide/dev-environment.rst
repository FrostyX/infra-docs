
=======================
Development Environment
=======================

In order to make contributing easy, all projects should have an automated way
to create a development environment. This might be as simple as a Python
virtual environment, or it could be a virtual machine or container. This
document provides guidelines for setting up development environments.

.. _note:
    You may encounter projects that do not yet have an automated development
    environment. Automating a project's development environment is a great way
    to get involved in a project and is incredibly helpful to your fellow
    contributors!


Ansible
=======

`Ansible`_ is used throughout Fedora Infrastructure to
automate tasks. If the project requires anything more than a Python virtual
environment to be set up, you should use Ansible to automate the setup.


Vagrant
=======

`Vagrant`_ is a tool to provision virtual machines. It allows you to define a
base image (called a "box"), virtual machine resources, network configuration,
directories to share between the host and guest machine, and much more. It can
be configured to use `libvirt`_ to provision the virtual machines.

You can install `Vagrant`_ on a Fedora host with::

    $ sudo dnf install libvirt vagrant vagrant-libvirt vagrant-sshfs

You can combine your `Ansible playbook`_ with `Vagrant`_ very easily. Simply
point Vagrant to your Ansible playbook and it will run it. Users who would
prefer to provision their virtual machines in some other way are free to do so
and only need to run the Ansible playbook on their host.

.. note::
    How a project lays out its development-related content is up to the
    individual project, but a good approach is to create a ``devel`` directory.
    Within that directory you can create an ``ansible`` directory and use the
    layout suggested in the `Ansible roles`_ documentation.

Below is a Vagrantfile that provisions a Fedora 25 virtual machine, updates it, and
runs an Ansible playbook on it. You can place it in the root of your repository as
``Vagrantfile.example`` and instruct users to copy it to ``Vagrantfile`` and
customize as they wish.

.. code-block:: ruby

   # -*- mode: ruby -*-
   # vi: set ft=ruby :
   #
   # Copy this file to ``Vagrantfile`` and customize it as you see fit.

   VAGRANTFILE_API_VERSION = "2"

   Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    # If you'd prefer to pull your boxes from Hashicorp's repository, you can
    # replace the config.vm.box and config.vm.box_url declarations with the line below.
    #
    # config.vm.box = "fedora/25-cloud-base"
    config.vm.box = "f25-cloud-libvirt"
    config.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/releases"\
                        "/25/CloudImages/x86_64/images/Fedora-Cloud-Base-Vagrant-25-1"\
                        ".3.x86_64.vagrant-libvirt.box"

    # Forward traffic on the host to the development server on the guest.
    # You can change the host port that is forwarded to 5000 on the guest
    # if you have other services listening on your host's port 80.
    config.vm.network "forwarded_port", guest: 5000, host: 80

    # This is an optional plugin that, if installed, updates the host's /etc/hosts
    # file with the hostname of the guest VM. In Fedora it is packaged as
    # ``vagrant-hostmanager``
    if Vagrant.has_plugin?("vagrant-hostmanager")
        config.hostmanager.enabled = true
        config.hostmanager.manage_host = true
    end

    # Vagrant can share the source directory using rsync, NFS, or SSHFS (with the vagrant-sshfs
    # plugin). Consult the Vagrant documentation if you do not want to use SSHFS.
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.synced_folder ".", "/home/vagrant/devel", type: "sshfs", sshfs_opts_append: "-o nonempty"

    # To cache update packages (which is helpful if frequently doing `vagrant destroy && vagrant up`)
    # you can create a local directory and share it to the guest's DNF cache. Uncomment the lines below
    # to create and use a dnf cache directory
    #
    # Dir.mkdir('.dnf-cache') unless File.exists?('.dnf-cache')
    # config.vm.synced_folder ".dnf-cache", "/var/cache/dnf", type: "sshfs", sshfs_opts_append: "-o nonempty"

    # Comment this line if you would like to disable the automatic update during provisioning
    config.vm.provision "shell", inline: "sudo dnf upgrade -y"

    # bootstrap and run with ansible
    config.vm.provision "shell", inline: "sudo dnf -y install python2-dnf libselinux-python"
    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "devel/ansible/vagrant-playbook.yml"
    end

    # Create the "myproject" box
    config.vm.define "myproject" do |myproject|
       myproject.vm.host_name = "myproject.example.com"

       myproject.vm.provider :libvirt do |domain|
           # Season to taste
           domain.cpus = 4
           domain.graphics_type = "spice"
           domain.memory = 2048
           domain.video_type = "qxl"

           # Uncomment the following line if you would like to enable libvirt's unsafe cache
           # mode. It is called unsafe for a reason, as it causes the virtual host to ignore all
           # fsync() calls from the guest. Only do this if you are comfortable with the possibility of
           # your development guest becoming corrupted (in which case you should only need to do a
           # vagrant destroy and vagrant up to get a new one).
           #
           # domain.volume_cache = "unsafe"
       end
    end
   end


.. _Ansible: https://www.ansible.com/
.. _Ansible roles: https://docs.ansible.com/ansible/playbooks_roles.html
.. _Ansible playbook: https://docs.ansible.com/ansible/playbooks.html
.. _Vagrant: https://www.vagrantup.com/
.. _libvirt: https://libvirt.org
