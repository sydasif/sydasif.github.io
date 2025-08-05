---
title:  "How to Connect Linux Host PC (Ubuntu) with GNS3"
date:   2022-05-21 15:02:15 +0500
categories: linux
tags:
  - gns3
  - lab
  - linux bash
---

## How to install a tap0 interface in Linux for GNS3
After installation of GNS3, we will install loopback adapter on our Ubuntu systems, so that we can connect to devices's in GNS3 from our host OS.

### Loopback tap installation on Ubuntu

```console
user@admin-desktop:~$ sudo –i
root@ admin-desktop:~#apt-get install uml-utilities
root@ admin-desktop:~#modprobe tun
root@ admin-desktop:~#tunctl
root@ admin-desktop:~#ifconfig tap0 10.10.10.10 netmask 255.255.255.0 up
```

> Verify that tap0 is up and given ip is assigned.
{: .prompt-tip }

```console
root@ubuntu:~# ifconfig
tap0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.10.10.10  netmask 255.255.255.0  broadcast 10.10.10.255
        ether 62:50:c1:1e:4e:c7  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

If you want to add one more loopback interface, This will create loopback interface tap1 on your device.

```console
root@ admin-desktop:~#tunctl
```

Change the ip with ifconfig according to your requirement.

Run GNS3 with root privileges, or you won’t be able to add these tap interfaces to GNS3.

Drag the cloud onto the work-board and right click on it, Select Configure. Then Select NIO TAP, type tap0, press Add. Click Apply & ok. Now connect it to your router.
