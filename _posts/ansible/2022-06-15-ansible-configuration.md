---
title:  "Ansible Configuration and Inventory Setup: A Comprehensive Guide"
date:   2022-06-15 14:05:00 +0500
categories: ansible
description: "Learn how to configure Ansible and set up an inventory file for network automation. This guide covers the ansible.cfg file, inventory formats, and best practices."
tags:
  - ansible
  - configuration
  - inventory
  - network automation
  - ansible.cfg
  - hosts file
---

## [Ansible Configuration and Creating Inventory File](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html#configuring-ansible)
Before we start using Ansible for network automation, there is one more step we need to prepare Ansible. That is preparing Ansible configuration file and inventory file.

### Configuration file

Certain settings in Ansible are adjustable via a configuration file _(ansible.cfg)_. The default configuration should be sufficient, but there may be reasons you would want to change them. Ansible configuration file path is searched in the following order:

- ANSIBLE_CONFIG (environment variable if set)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /etc/ansible/ansible.cfg

Ansible will process the above list and use the first file found, all others are ignored.

```console
root@NetworkAutomation-1:~# ansible --version
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

When we use the ansible --version command as above, it will show the exact location of the configuration file, which by default is in /etc/ansible/ansible.cfg, first line shows the location of ansible configuration file.

If we filter the contents of the ansible configuration file to show only the inventory file, we see that the inventory file is located in /etc/ansible/hosts by default. You can specify a different inventory file at the command line using the -i _path-to-hosts-file_ option. You can also use multiple inventory files at the same time, or different formats (YAML, ini, etc). We add to the inventory file all the devices that we want to manage them.

```terminal
root@NetworkAutomation-1:~# cat /etc/ansible/ansible.cfg | grep inventory

#inventory      = /etc/ansible/hosts
```

Fortunately, you can overrides both default Ansible configuration file and default inventory file for each project, since each project may have its own configuration parameters and a different list of devices to be managed remotely.

It is therefore recommended to create a folder with its own configuration and inventory file for each project.

In ansible.cfg we define the location of the inventory file which will be used for this project and some other basic parameters as follows.

```console
[defaults]
inventory = ./hosts
timeout = 60
host_key_checking = false
```

To check ansible configuration file we use the command.

```console
root@NetworkAutomation-1:~# ansible-config view
[defaults]
inventory = ./hosts
timeout = 60
host_key_checking = false
```

And to check the configurations that are different from default, we use the command as below:

```console
root@NetworkAutomation-1:~# ansible-config dump --only-changed
DEFAULT_HOST_LIST(/root/ansible.cfg) = ['/root/hosts']
DEFAULT_TIMEOUT(/root/ansible.cfg) = 60
HOST_KEY_CHECKING(/root/ansible.cfg) = False
```

## [How to build your inventory](https://docs.ansible.com/ansible/2.9/user_guide/intro_inventory.html)

In the hosts inventory file we add the names or IP of the devices that are to be managed remotely.Preferably use the name of the devices instead of the IP addresses and make sure that these names can be resolved via the DNS server or from the local DNS file.

So first we update /etc/hosts (local DNS file) with IP and hostname, and then we create the inventory file.

```console
root@NetworkAutomation-1:~# cat /etc/hosts
127.0.1.1       NetworkAutomation-1
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.10.10     sw1
192.168.10.20     r1
192.168.10.30     r2
```

### Inventory

- Inventory is the target host in our infrastructure which we want to manage by ansible. We put all the hosts information (IP Address/FQDN) into one file called inventory.
- The default inventory file is /etc/ansible/hosts.
- Inventory is a file in INI or YAML format.
- The headings in brackets are group names, which are used in classifying hosts.
- There are two default groups, all and ungrouped, all contain every host, ungrouped contains all hosts that do not have any other group aside from all.

Now let’s create an inventory hosts file. In the first step we enter the name of device's into group and then add groups to the network group and lastly, variables as below:

```console
root@NetworkAutomation-1:~# cat hosts
[switch]
sw1

[router]
r1
r2

[network:children]
switch
router

[network:vars]
ansible_user=admin
ansible_password=cisco
ansible_network_os=ios
ansible_connection=network_cli
```

Then we use the following commands to ensure that the configuration file and hosts file defined for this folder are used for the ansible and not the original ones.

```console
root@NetworkAutomation-1:~# ansible --version
ansible 2.9.9
  config file = /root/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

```console
root@NetworkAutomation-1:~# ansible --list-hosts all
  hosts (3):
    sw1
    r1
    r2
```

You can see that the correct configuration and host files are being used for this lab.

> Now we test our lab with ping module, the result is ok.

```json
root@NetworkAutomation-1:~# ansible all -m ping
r2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
r1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
sw1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
