# Quality Infrastructure Development Environments

This repository contains tools that can automatically provision a standardized development environment. Using these tools also ensures that the Prosperity4All Quality Infrastructure can automatically build and test your software. With this approach it is possible to:

* Spin up VMs containing application stacks using simple commands
* Sync file system changes from your operating system (also referred to as 'the host') to the VM so native development tools like editors and IDEs can be used
* Forward ports programmatically from the host to the VM
* Define entire development environments alongside application source code
* Treat VMs as disposable development or test environments since they can be reprovisioned with minimal effort
* Utilize [CentOS](https://github.com/idi-ops/packer-centos) and [Fedora](https://github.com/idi-ops/packer-fedora) VMs that are kept upto date with security patches by the Inclusive Design Institute

## Requirements

The following software needs to be installed on the host OS:

* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [Vagrant](https://www.vagrantup.com/downloads.html)
* OpenSSH client

**Note:** Windows support was tested using a [Cygwin](https://cygwin.com) shell after having performed the following tasks:

* Installed OpenSSH using Cygwin
* Made sure the Windows firewall was not restricting Vagrant or VirtualBox

## Contents

Each subdirectory contains a [Vagrantfile](http://docs.vagrantup.com/v2/vagrantfile/) and a ``provisioning`` directory. The directory contains the following files:

* ``playbook.yml`` - an [Ansible playbook](http://docs.ansible.com/ansible/playbooks.html) that orchestrates the provisioning process
* ``requirements.yml`` - specifies the [Ansible roles](http://docs.ansible.com/ansible/playbooks_roles.html) that the playbook requires
* ``vars.yml`` - a list of variables used by the playbook, this is the only file that needs to be modified to get started

Currently only Node.js applications are supported. To get started you should copy the contents of the ``nodejs`` directory (excluding the README) to the top-level of your application's source code tree. You will want to edit the ``provisioning/vars.yml`` file. With these changes in place you can type ``vagrant up`` in the location where you copied the ``Vagranfile`` to which will boot the VM. 

## Notable Vagrant Commands

While the [Vagrant web site](https://docs.vagrantup.com/v2/cli/index.html) provides extensive documentation these are the bare minimum commands that you should ideally be familiar with:

* ``vagrant up`` - start up a new (provisioning takes place when starting new VMs) or stopped VM
* ``vagrant provision`` - trigger the provisioning process again at any given time
* ``vagrant ssh`` - log into the VM using SSH
* ``vagrant status`` - verify whether the VM has been provisioned once, if it's running, or stopped
* ``vagrant halt`` - stop the VM
* ``vagrant destroy`` - delete the VM altogether

For convenience sake it is suggested that the following shell aliases get added to your ``~/.bashrc`` or ``~/.zshrc`` file:

    alias vup="vagrant up"
    alias vpr="vagrant provision"
    alias vss="vagrant ssh"
    alias vst="vagrant status"
    alias vha="vagrant halt"

## Logs

If a Node.js application's VM is launched its logs can be viewed using the following URL:
[http://127.0.0.1:19531/entries?_EXE=/usr/bin/node&follow](http://127.0.0.1:19531/entries?_EXE=/usr/bin/node&follow)

The result will be a stream of log events generated by the ``/usr/bin/node`` process. The stream will autoupdate as new events are logged in the VM since the ``follow`` parameter is included in the URL.

Log events are exposed this way by having a small [HTTP server](http://www.freedesktop.org/software/systemd/man/systemd-journal-gatewayd.service.html) act as a gateway for the VM's [Systemd journal](https://wiki.archlinux.org/index.php/Systemd#Journal). The journal can be [queried](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs) with command line tools such as [journalctl](http://www.freedesktop.org/software/systemd/man/journalctl.html):

    vagrant ssh -c 'sudo journalctl -f /usr/bin/node'

## Port Forwarding

The provided Vagrantfile(s) will forward TCP ports from the host to the VM. Ports specific to the application should be documented in the ``provisioning/vars.yml`` configuration but defaults are also included in the Vagrantfile itself. This behaviour can be overridden by setting a ``VM_HOST_TCP_PORT`` environment variable before using the ``vagrant up`` command. For example:

    VM_HOST_TCP_PORT=31337 vagrant up

If two VMs requiring the same ports are launched and no ``VM_HOST_TCP_PORT`` environment variables are used to prevent a conflict, then the second VM will attempt to correct this. A message such as the following will be logged to your terminal:

    Fixed port collision for 8080 => 8080. Now on port 2200.

