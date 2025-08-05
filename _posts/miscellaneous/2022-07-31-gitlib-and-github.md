---
title:  "Manage GitHub and Gitlab accounts on a single machine"
date:   2022-07-31 18:20:15 +0500
categories: miscellaneous
tags:
  - git
  - github
  - gitlab
---

## Manage GitHub and Gitlab accounts on a single machine with SSH keys on Linux
For this example, We will use the same email address to produce two different SSH keys:

- one for Github, and
- the other is for Gitlab

The logic also extends to different emails, and multiple Github, and Gitlab accounts.

## Generate new SSH keys

Check if you have any existing SSH keys with the `ls -al ~/.ssh` command on your machine. You can also use the existing key pair if available, but I suggest you don’t.

To create private and public SSH pairs for your personal, and work accounts, run the commands:

```console
ssh-keygen -t rsa -C "personal@mail.com" -f ~/.ssh/id_rsa_github
ssh-keygen -t rsa -C "personal@mail.com" -f ~/.ssh/id_rsa_gitlab
```

The above commands will produce four different files namely as below:

```console
~/.ssh/id_rsa_github
~/.ssh/id_rsa_github.pub
~/.ssh/id_rsa_gitlab
~/.ssh/id_rsa_gitlab.pub
```

## Add SSH keys to Github and Gitlab

### Github

Copy the corresponding public key of your Gitlab account

```console
cat ~/.ssh/id_rsa_github.pub
```

Log on to your Github account. Then go to `Settings > SSH and GPG Keys > New SSH key`

Paste the public key on the Key section, and edit the title.

### Gitlab

Copy the corresponding public key of your Gitlab account

```console
cat ~/.ssh/id_rsa_gitlab.pub
```

Log on to your Gitlab account. Then go to `Settings > SSH Keys`

Paste the public key in the key window, and edit the title.

## Register SSH Keys with the ssh-agent

Register all SSH keys on your local machine using the ssh-agent

```console
ssh-add ~/.ssh/id_rsa_github
ssh-add ~/.ssh/id_rsa_gitlab
```

## Editing Config File

In the last step, create a configuration file that will add the different SSH keys for all online repositories and emails we created earlier.

```console
touch ~/.ssh/config
vim ~/.ssh/config
```

The config file should look like this

```console
# Personal account
Host github.com
   HostName github.com
   IdentityFile ~/.ssh/id_rsa_github# Personal gitlab account

# Work account
Host gitlab.com
   HostName gitlab.com
   IdentityFile ~/.ssh/id_rsa_gitlab
```

After following the above steps, you should be able to clone and edit both your GitHub and GitLab repositories on your local machine.
