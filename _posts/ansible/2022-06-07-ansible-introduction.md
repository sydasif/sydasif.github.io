---
title:  "Part 01: Ansible for Network Automation"
date:   2022-06-07 15:14:00 +0500
categories: ansible
tags:
  - ansible
toc: true
toc_label: "On This Post"
toc_sticky: true
---

## What is Ansible?
Ansible is a configuration management system. Ansible allows you to automate and simplify the configuration, maintenance, and deployment of servers, services, software, and more.

- Ansible is an open-source, similar to Chef, Puppet, and Salt.
- Ansible lets you control and configure nodes from a single machine.
- SSH connection to connect to devices and run the configuration task.
- parallel connection to devices via SSH

### Why Ansible?

- *Simple:* Ansible uses a simple syntax written in *YAML* format called playbook.
- *Agent-less:* There are no agent/software or additional firewall ports that you need to install on the client’s systems or hosts which you want to automate.
- *Powerful and Flexible:* Ansible has powerful features that enable you to model even the most complex IT workflow.
- *Efficient:* No extra software on your server, which means more resources for your applications.
- *Idempotent:* Ansible will not change the device's state unless it needs to change its setting to reach the desired state.

## [Ansible for Network Automation](https://docs.ansible.com/ansible/2.9/network/index.html)

As with other automation tools, Ansible started out by managing servers before expanding its ability to manage networking equipment. For the most part, the modules and what Ansible refers to as the 'playbooks' are similar between server modules and network modules, with some differences.

Ansible is actively developing towards support for network equipment, and new features and modules for working with network equipment are constantly appearing in it.
