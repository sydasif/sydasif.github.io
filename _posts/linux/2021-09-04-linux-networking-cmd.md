---
title:  "Linux Networking Commands"
date:   2021-09-04 15:02:15 +0500
categories: linux
tags:
  - commands
  - linux bash
---

## Linux Networking Commands
In this memo, I'm showing Linux networking commands to remember as a network engineer/technician.

To show the IP address assigned to an interface:

```console
ip address show
```

To assign an IP  Address to an interface, for example, ens33:

```console
ip address add 192.168.8.20/24 dev ens33
```

To delete an IP Address on an interface:

```console
ip address del 192.168.8.20/24 dev ens33
```

To online or up an interface, for example, eth0:

```console
ip link set eth0 up
```

To offline or down an interface:

```console
ip link set eth0 down
```

To change an interface MTU, the default MTU is 1500:

```console
ip link set eth0 mtu 9000
```

To add a default route:

```console
ip route add default via 192.168.1.254 dev eth0
```

To add a route to 192.168.1.0/24 via the gateway at 192.168.1.254:

```console
ip route add 192.168.1.0/24 via 192.168.1.254
```

Add a route to 192.168.1.0/24 that can be reached on the device:

```console
ip route add 192.168.1.0/24 dev ens33
```

Delete the route for 192.168.1.0/24 via the gateway at 192.168.1.254:

```console
ip route delete 192.168.1.0/24 via 192.168.1.254
```
