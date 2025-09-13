---
title:  "Protecting Sensitive Data with Ansible Vault: A Practical Guide"
date:   2022-06-22 09:55:00 +0500
categories: ansible
description: "Learn how to use Ansible Vault to encrypt sensitive data, such as passwords and API keys, in your Ansible playbooks and inventory files. This guide provides practical examples and best practices for securing your Ansible deployments."
tags:
  - ansible
  - vault
  - encryption
  - security
  - network automation
  - passwords
  - ansible-vault
---

## [Protecting sensitive variables with vault](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html#protecting-sensitive-variables-with-ansible-vault)

The ansible-vault command provides encryption for files and/or individual variables like passwords. We can use the commands below to encrypt sensitive variable string information:

We prefer to type our ansible-vault password rather than store it in a file, so we can request a prompt:

```console
ansible-vault encrypt_string --vault-id id_01@prompt 'cisco' --name 'ansible_password'
New vault password (id_01):
Confirm new vault password (id_01):
ansible_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;id_01
          38643238356131393038366634663031313834343262663637633332323533623030303634353235
          3631626636353035343764613737323235646363396263630a353630343332336231396465353265
          31666235323066353331333063653239636238363864376134306639393633306235373166616135
          3131323232626132610a623463336338383739663634313861653237653661656464656263623332
          3233
Encryption successful
```

Now we copy the above output of ansible-vault command to our YAML inventory host file as below:

```yml
---
  network:
    vars:
      ansible_user: admin
      ansible_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;id_01
          38643238356131393038366634663031313834343262663637633332323533623030303634353235
          3631626636353035343764613737323235646363396263630a353630343332336231396465353265
          31666235323066353331333063653239636238363864376134306639393633306235373166616135
          3131323232626132610a623463336338383739663634313861653237653661656464656263623332
          3233
      ansible_network_os: cisco.ios.ios
      ansible_connection: ansible.netcommon.network_cli
    children:
      switch:
        hosts:
          access:
      dist:
        hosts:
          dist-1:
          dist-2:
```

The --vault-id flag allows different vault passwords for different users or different levels of access.

Now we create a file and write our password for ansible-vault to it and pulling ansible-vault password from the file with below ansible-vault command:

```terminal
ansible-playbook --vault-id id_01@~/path/to/file 01_playbook.yml
```

or we can also use below command to enter vault password at prompt.

```terminal
ansible-playbook --vault-id id_01@prompt 01_playbook.yml
```

[see more on ansible vault](https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data)
