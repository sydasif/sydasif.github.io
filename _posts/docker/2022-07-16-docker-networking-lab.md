---
title:  "Part 02: Docker for Network Automation"
date:   2022-07-16 07:54:00 +0500
categories: docker
tags:
  - python3
  - network automation
  - docker
---

## Docker networking  overview lab
In Docker networking, we will learn some basic concepts and prepares you to design and deploy your applications to take full advantage of these capabilities.

### [Networking overview](https://docs.docker.com/network/)

One of the reasons Docker containers and services are so powerful is that you can connect them together, or connect them to non-Docker environment. Docker containers and services do not even need to be aware that they are deployed on Docker, or whether their peers are also Docker workloads or not. Whether your Docker hosts run Linux, Windows, you can use Docker to manage them.

### Network drivers

Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality:

```console
╭─$ ~
╰─➤  docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
b26a10be36c4   bridge    bridge    local
581c78f06712   host      host      local
d806b918ebca   none      null      local
```

- **bridge:** This is the default network driver. If you don’t specify a driver, this is the type of network you are creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate.

- **host:** For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly.

- **none:** For this container, disable all networking. Usually used in conjunction with a custom network driver.

Some other network drivers are below:

- overlay
- ipvlan
- macvlan

### Network driver summary

- User-defined bridge networks are best when you need multiple containers to communicate on the same Docker host.
- Host networks are best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated.
- Overlay networks are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.
- Macvlan networks are best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address.

## [Networking tutorials](https://docs.docker.com/network/#networking-tutorials)

Now that you understand the basics about Docker networks, deepen your understanding using the following tutorials:

- Standalone networking tutorial

### Networking with standalone containers

This tutorials deals with networking for standalone Docker containers.

#### Use the default bridge network

Use the default bridge network demonstrates how to use the default bridge network that Docker sets up for you automatically. This network is not the best choice for production systems.

Open a terminal, list current networks. Here’s what you should see if you’ve never added a network or initialized a swarm on this Docker daemon. You may see different networks, but you should at least see these (the network IDs will be different):

```console
╭─$ ~
╰─➤  docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
b26a10be36c4   bridge    bridge    local
581c78f06712   host      host      local
d806b918ebca   none      null      local
```

The default bridge network is listed, along with host and none. The latter two are not fully-fledged networks, but are used to start a container connected directly to the Docker daemon host’s networking stack, or to start a container with no network devices. This tutorial will connect two containers to the bridge network.

In this tutorial, we start two different alpine containers on the same Docker host and do some tests to understand how they communicate with each other. You need to have Docker installed and running.

```console
docker run -dit --name alpine1 alpine ash

docker run -dit --name alpine2 alpine ash
```

We start two alpine containers running _ash_, which is Alpine’s default shell rather than _bash_. The _-dit_ flags mean to start the container in the background, interactive (with the ability to type into it), and with a TTY (so you can see the input and output). We won’t be connected to the container right now. Instead, the container’s ID will be printed and the containers connect to the default _bridge_ network.

Check that both containers are actually started:

```console
╭─$ ~
╰─➤  docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS              PORTS     NAMES
d9e5a4cd93b2   alpine    "ash"     9 seconds ago        Up 6 seconds                  alpine2
ae411e3baa20   alpine    "ash"     About a minute ago   Up About a minute             alpine1
```

Inspect the bridge network to see what containers are connected to it.

```console
╭─$ ~
╰─➤  docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b5dc172221123bb4d29fdfed1e82a622ac16733c85d7c036012b0276a50700ac",
        "Created": "2022-07-20T07:36:16.077416079+05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "ae411e3baa20cb065aa1c9372011508cad605f0c6e3109d4343ac91367d49221": {
                "Name": "alpine1",
                "EndpointID": "e295680e94a406852cc08e9cbb6121b412cf61864ed377fb9cd7625e7eeca1e7",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "d9e5a4cd93b2fb7332effeb5bebd252493e93c189a98e51836e181c81cf5101b": {
                "Name": "alpine2",
                "EndpointID": "aec95ea451c080ad87e1e58b03cae98b620cb9429a697937ebeca29cbd3a20db",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

The information about the bridge network is listed, including the IP address of the gateway between the Docker host and the bridge network (172.17.0.1). Under the Containers key, each connected container is listed, along with information about its IP address (172.17.0.2 for alpine1 and 172.17.0.3 for alpine2).

The containers are running in the background. Use the docker attach command to connect to alpine1.

```console
╭─$ ~
╰─➤  docker attach alpine1
/ #
```

The prompt changes to # to indicate that you are the root user within the container. Use the ip addr show command to show the network interfaces for alpine1 as they look from within the container:

```console
/ # ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

Notice that the second interface has the IP address 172.17.0.2, which is the same address shown for alpine1 in the previous step.

Now try to ping the second container. First, ping it by its IP address, 172.17.0.3:

```console
/ # ping 172.17.0.3 -c 3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.160 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.234 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.267 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.160/0.220/0.267 ms
```

Now exit from alpine1 without stopping it by using, _CTRL + pq_ (hold down CTRL and type p followed by q). If you wish, attach to alpine2 and repeat the procedure, to check ping from alpine2.

Stop and remove both containers.

```console
docker container stop alpine1 alpine2

docker container rm alpine1 alpine2
```

#### Use user-defined bridge networks

Create the alpine-net network. You do not need the --driver bridge flag since it’s the default, but this example shows how to specify it.

```console
docker network create --driver bridge alpine-net
```

List Docker’s networks:

```console
╭─syd@ubuntu ~
╰─➤  docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
c4469ec38b8e   alpine-net   bridge    local
b5dc17222112   bridge       bridge    local
581c78f06712   host         host      local
d806b918ebca   none         null      local
```

Inspect the alpine-net network. This shows you its IP address and the fact that no containers are connected to it:

```console
╭─syd@ubuntu ~
╰─➤  docker network inspect alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "c4469ec38b8e9959cdc0cb89929b47df743d4b786f7549f44fe6caed0b9873ca",
        "Created": "2022-07-20T08:23:44.290323737+05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

Notice that this network’s gateway is 172.18.0.1, as opposed to the default bridge network, whose gateway is 172.17.0.1. The exact IP address may be different on your system.

Create your two containers. Notice the _--network_ flags. You can only connect to one network during the docker run command.

```console
docker run -dit --name alpine1 --network alpine-net alpine ash
docker run -dit --name alpine2 --network alpine-net alpine ash
```

Create one more container and use default network:

```console
docker run -dit --name alpine3 alpine ash
```

Verify that all containers are running:

```console
╭─$ ~
╰─➤  docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS         PORTS     NAMES
47b8af26985d   alpine    "ash"     11 seconds ago   Up 8 seconds             alpine3
6407554a1a44   alpine    "ash"     3 minutes ago    Up 3 minutes             alpine2
245f3ed32afd   alpine    "ash"     3 minutes ago    Up 3 minutes             alpine1
```

Inspect the bridge network and the alpine-net network again:

```console
╭─$ ~
╰─➤  docker network inspect bridge                                                                             130 ↵
[
    {
        "Name": "bridge",
        "Id": "b5dc172221123bb4d29fdfed1e82a622ac16733c85d7c036012b0276a50700ac",
        "Created": "2022-07-20T07:36:16.077416079+05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "47b8af26985d7194f9376f908b62df3514a626ed1bf4cf260c58602410df4a96": {
                "Name": "alpine3",
                "EndpointID": "5a672b00ab78ad8d285563b13876777b6d822e96c8497b5ddb2239aefc2786f3",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

Containers alpine3 is connected to the bridge network.

```console
╭─$ ~
╰─➤  docker network inspect alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "c4469ec38b8e9959cdc0cb89929b47df743d4b786f7549f44fe6caed0b9873ca",
        "Created": "2022-07-20T08:23:44.290323737+05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "245f3ed32afd645c80fe3a99d43c899451d2698beacf8c7b3c7b0cacadb90bb4": {
                "Name": "alpine1",
                "EndpointID": "81baac80ab7d068e2849e5afceb74f52e5c80f159dfe8e91591ea341da070ac1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "6407554a1a440c15de3ac843999e76a43a5a654ebd76eeadf70e95ea49225295": {
                "Name": "alpine2",
                "EndpointID": "92a9dfbcdb101c169d464bf90f396e37c5bc4508e1d53d0574b19b9d5ef84cd8",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

Containers alpine1 and alpine2 are connected to the alpine-net network.

On user-defined networks like alpine-net, containers can not only communicate by IP address, but can also resolve a container name to an IP address. This capability is called automatic service discovery. Let’s connect to alpine1 and test this out. alpine1 should be able to resolve alpine2 to IP addresses.

```console
╭─$ ~
╰─➤  docker container attach alpine1
/ # ping alpine2 -c 3
PING alpine2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.252 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.192 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.193 ms

--- alpine2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.192/0.212/0.252 ms
```

From alpine1, you should not be able to connect to alpine3 at all, since it is not on the alpine-net network.

```console
/ # ping alpine3 -c 3
ping: bad address 'alpine3'

/ # ping  172.17.0.2 -c 3
PING 172.17.0.2 (172.17.0.2): 56 data bytes

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
```

From alpine1, you should not be able to connect to alpine3 at all, since it is not on the alpine-net network. Now exit from alpine1 without stopping it by using, _CTRL + pq_.

Now we, create one more container and connect it to both network as below. It should be able to reach all of the other containers. However, you will need to ping alpine3 by its IP address.

```console
╭─$ ~
╰─➤  docker run -dit --name alpine4 --network alpine-net alpine ash
9ac444064e99eddc19940162ed48405c9434b6aa5871d2b332d7f8c8e98dc08e

╭─$ ~
╰─➤  docker network connect bridge alpine4
```

Attach to alpine4 and run the tests.

```console
╭─syd@ubuntu ~
╰─➤  docker container attach alpine4
/ # ping alpine1 -c 3
PING alpine1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.285 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.197 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.201 ms

--- alpine1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.197/0.227/0.285 ms
/ # ping alpine2 -c 3
PING alpine2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.239 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.186 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.200 ms

--- alpine2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.186/0.208/0.239 ms
/ # ping 172.17.0.2 -c 3
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.274 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.195 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.161 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.161/0.210/0.274 ms
```

> As a final test, make sure your containers can all connect to the internet by pinging google.com.

Now exit from alpine4 without stopping it by using, _CTRL + pq_ and stop and remove all containers and the alpine-net network.

```console
docker container stop alpine1 alpine2 alpine3 alpine4

docker container rm alpine1 alpine2 alpine3 alpine4

docker network rm alpine-net
```

Now that we have completed the networking tutorials for standalone containers. In the next post, we will show you how to connect our container with a network in GNS3 and run the netmiko script.

**Source:** <https://docs.docker.com/network/>
