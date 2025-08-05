---
title:  "Part 06: Ansible Multi version Installation"
date:   2022-06-17 09:55:00 +0500
categories: ansible
tags:
  - automation
  - python
toc: true
toc_label: "On This Post"
toc_sticky: true
---

## Ansible and Python3 venv module
This is a module shipped by Python3 to create a virtual environment, it creates and used python3 to build the environment. we are going to use this module and deploy multiple Ansible version on same control node, so that even if we have to contact the managed hosts with old python version or for some deprecated feature or any latest one, we are not bounded to deploy a separate control node.

Also it provide leverage to test newer Ansible release for compatibility, without need to upgrade the actual Ansible setup.

## What is virtual environment or venv ?

These are python tools/modules used for creating lightweight “virtual environments”. Each virtual environment has its own Python binary (which matches the version of the binary that was used to create this environment) and can have its own independent set of installed Python packages in its site directories.

Let’s deploy some other versions of ansible by python3-venv module:

```terminal
sudo apt install python3-venv
```

Make a directory and changed it to that directory and create a virtual environment.

```terminal
python3 -m venv ansible-2.7
```

Activate your virtual environment:

```terminal
source ansible-2.7/bin/activate
```

Upgrade pip3 and Install wheel and paramiko:

```terminal
pip install --upgrade pip
pip3 install wheel paramiko
```

Now install ansible 2.7

```terminal
pip3 install ansible==2.7
```

```terminal
deactivate
```

Deactivate virtual environment and repeat the same commands to deploy the ansible 2.9 version.
