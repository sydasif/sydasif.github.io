---
title:  "SSH Error - Unable to Negotiate Legacy Algorithm and Cipher"
date:   2021-09-23 11:50:00 +0500
categories: networking
tags:
  - gns3
  - networking
  - SSH
  - legacy algorithm
  - legacy cipher
  - cisco
  - Windows
  - Linux
  - troubleshooting
  - network devices
  - network protocols
  - SSH configuration
  - SSH troubleshooting
---

## Legacy SSH Algorithm and Cipher in Cisco Device's
In this blog, I'm showing you how to enable the legacy SSH algorithm and cipher option on your machine.

I, encounter this issue so many times whenever setting lab environment for testing. I'm using Windows 10 and Microsoft loopback adaptor to connect with devices in GNS3, but this process is also working on any other setup or any Linux OS.

Lab Topology used in this setup as shown below:

![lab-layout](/assets/img/ssh_issu_lab.png)

First, configure devices for SSH and assigned IP addresses as per your lab environment. When I, SSH into a device found the below error:

```console
$ ssh admin@9.9.9.11

Unable to negotiate with 9.9.9.11 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
```

To set up this error on a permanent, add a file named _config_ in your home directory in the .ssh folder. After that copy the algorithm name and add KexAlgorithms +diffie-hellman-group1-sha1 into the config file and host IP as below:

```console
Host 9.9.9.11
 KexAlgorithms +diffie-hellman-group1-sha1
```

After that  I have another issue found as below:

```console
No matching ciphers found. aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc
```

Now copy the ciphers from the terminal and add them to the config file under host as below:

```console
Host 9.9.9.11
 KexAlgorithms +diffie-hellman-group1-sha1
 Ciphers aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc
```

You can use check this **blog** for any other [algorithms and ciphers](https://www.infosecmatter.com/solution-for-ssh-unable-to-negotiate-errors/) in any OS.
