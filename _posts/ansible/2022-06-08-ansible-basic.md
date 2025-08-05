---
title: "Part 02: Getting Started with Ansible"
date: 2022-06-08 08:14:00 +0500
categories: ansible
tag: ansible
toc: true
toc_label: "On This Post"
toc_sticky: true
---

## Ansible Basic and Installation
Ansible modules support a wide range of vendors, device types, and actions, so you can manage your entire network with a single automation tool. Ansible handles communication between control node and managed nodes through multiple protocols:

- network_cli by SSH
- netconf by SSH
- httpapi by HTTP/HTTPS

### Network platforms supported by Ansible

- Arista: eos
- Cisco: ios, iosxr, nxos
- Juniper: junos
- VyOS: vyos

### Ansible can

- Works without installing an agent on managed hosts
- Uses SSH to connect to managed hosts
- Performs changes using Python modules that run on managed hosts
- Can perform actions locally on the management host
- Uses YAML to describe scenarios

### Ansible component

- **Control node:** A central management of Ansible and Ansible code runs on it and gathers information about the managed node.
- **Managed nodes:** Managed nodes are network devices that are managed by the control node.
- **Inventory:** This file describes hosts, groups of hosts, and variables that can also be created
- **Playbooks:** Playbooks Contains a list of actions that can be in the playbook.
- **Tasks:** Tasks are actions that running by Ansible.
- **Module:** Ansible module. Implements certain functions

## Installation

Ansible can be installed on most Unix/Linux systems, with the only dependency on Python 2.7 or Python 3.5. Currently, the Windows operating system is not officially supported as the control machine. For installation, you can use Ansible [website](https://docs.ansible.com/ansible/2.9/installation_guide/index.html), which supports various kinds of platforms. We will be installing Ansible on the Ubuntu machine.

```console
sudo apt update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get install ansible
```

### Installing Ansible with pip in GNS3 Network-automation

In general, it is recommended to install Ansible based on Python3 to get most of the functions of Ansible. To do this, uninstall the currently Ansible version, and replace it with Ansible based on Python3.

With the _apt remove_ command, we can remove the installed version of Ansible.

```console
apt remove ansible
apt --purge autoremove
```

Install pip3, and then Ansible over pip3.

```console
apt update
apt install python3-pip
pip3 install ansible==2.9.9
```

To test Ansible Installation, run the below commands:

```console
ansible --version
```

If we use _ansible --version_ after successful installation with pip3, it cannot find Ansible in our system. We use the find command to find Ansible’s location.

```console
root@NetworkAutomation-1:~# find / -name ansible | grep bin
/usr/local/bin/ansible
/gns3volumes/usr/local/bin/ansible
```

As you can see, it’s located in `/usr/local/bin/ansible`

Now we run the Ansible command to see the version of the installed Ansible.

```console
root@NetworkAutomation-1:~# /usr/local/bin/ansible --version
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

I will add Ansible executable folder to the system PATH directory. So we don’t have to switch to this folder every time.

```console
root@NetworkAutomation-1:~# export PATH=$PATH:/usr/local/bin/
root@NetworkAutomation-1:~# ansible --version
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

To permanently add this path to the system PATH, we will update the .bashrc file.

```console
nano ~/.bashrc
export PATH="$PATH:/usr/local/bin"
source ~/.bashrc
```

Now, we test Ansible with the ping module:

```console
root@NetworkAutomation-1:~# ansible localhost -m ping
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
