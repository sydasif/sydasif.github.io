---
title: "Netlab Tutorial: Build a Campus Network Lab with VLANs, VRRP, and OSPF"
date: 2025-09-04 17:04:00 +0500
description: "Learn how to build a campus-style network lab using Netlab and Containerlab with VLANs, VRRP, and OSPF. Step-by-step guide for network engineers."
categories: networking
tags:
- netlab tutorial
- containerlab network lab
- network automation lab
- campus network topology
- VLAN VRRP OSPF lab
- docker networking for labs
- isco network lab setup
- ansible network automation
- virtual network lab tools
- build reproducible labs
---

As a veteran network engineer, I have always relied on free tools for my labbing journey. I started with Packet Tracer from Cisco, then moved to [GNS3](https://www.gns3.com/) for network automation labs, which served me well for a long time. But integrating with other systems, like connecting external VMs, was not easy. [EVE-NG](https://www.eve-ng.net/) made that part simpler, and I enjoyed using it, but it demanded heavy resources.

These days I use [Containerlab](https://containerlab.dev/). It is lightweight, flexible, and works smoothly with Docker, so building and managing labs is much faster and less resource hungry. For automation and modern testing, it has become my main choice. Still, the real challenge in all these tools comes after the devices boot up — device provisioning. Spinning them up is easy, but preparing configs and making the devices usable always takes the most effort, to test network automation tools.

## Solving the Pain with _netlab_

**netlab** is an open-source tool that solves these issues by letting you define your topology at a high level in a simple YAML file. Instead of placing devices in a GUI or repeating base configs, **netlab** automates device provisioning by generating the Containerlab topology and configurations, including IPs, routing, and custom settings. It works with Containerlab or Vagrant, allowing you to spin up realistic labs in minutes and reproduce them anywhere with ease.

## [Getting Started with _netlab_](https://netlab.tools/install/#installing-python-package)

In this blog, we will leverage Containerlab with the **netlab** tool, and the requirements are:

* [Docker](https://www.docker.com/)
* [Containerlab](https://containerlab.dev/)
* [Container images](https://containerlab.dev/manual/kinds/cisco_iol/)
* [Ansible](https://docs.ansible.com/ansible/latest/index.html)

To install **netlab** on an existing system that already has the low-level tools installed, use:

```shell
python3 -m pip install networklab
```

For reference, I am running this on an Ubuntu 24.04 desktop with 16 GB of RAM and 4 vCPUs.

> If you face any issue with installation, see [Manual Virtual Machine Provisioning](https://netlab.tools/install/ubuntu-vm/#manual-virtual-machine-provisioning).
{: .prompt-tip }

### Defining a Topology

Let me share a basic topology and walk you through what will be created.

```yaml
# topology.yml
---
provider: clab
defaults:
  devices:
    iol.clab.image: vrnetlab/cisco_iol:17.12.01

nodes:
  R1:
    device: iol
  R2:
    device: iol
  R3:
    device: iol

links:
  - R1-R2-R3
```

This topology will do the following:

* Use Containerlab to deploy the nodes
* Define prefix information for different connections (**netlab** can do automatically)
* Specify the number of nodes
* Define how the nodes are connected

```terminal
❯ netlab up

┌──────────────────────────────────────────────────────────────────────────────────┐
│ CREATING configuration files                                                     │
└──────────────────────────────────────────────────────────────────────────────────┘
[CREATED] provider configuration file: clab.yml
[CREATED] transformed topology dump in YAML format in netlab.snapshot.yml
[GROUPS]  group_vars for all
[GROUPS]  group_vars for iol
[HOSTS]   host_vars for R1
[HOSTS]   host_vars for R2
[HOSTS]   host_vars for R3
[CREATED] minimized Ansible inventory hosts.yml
[CREATED] Ansible configuration file: ansible.cfg

┌──────────────────────────────────────────────────────────────────────────────────┐
│ CHECKING virtualization provider installation                                    │
└──────────────────────────────────────────────────────────────────────────────────┘
[SUCCESS] clab installed and working correctly

┌──────────────────────────────────────────────────────────────────────────────────┐
│ STARTING clab nodes                                                              │
└──────────────────────────────────────────────────────────────────────────────────┘
provider clab: executing sudo -E containerlab deploy --reconfigure -t clab.yml
16:18:14 INFO Containerlab started version=0.69.3
16:18:14 INFO Parsing & checking topology file=clab.yml
16:18:14 INFO Removing directory path=/home/zulu/Documents/demo-lab/clab-demolab
16:18:14 INFO Creating docker network name=netlab_mgmt IPv4 subnet=192.168.121.0/24 IPv6 subnet="" MTU=0
16:18:17 INFO Creating lab directory path=/home/zulu/Documents/demo-lab/clab-demolab
16:18:18 INFO Creating container name=R1
16:18:18 INFO Creating container name=R3
16:18:18 INFO Creating container name=R2
16:18:21 INFO Running postdeploy actions for Cisco IOL 'R1' node
16:18:21 INFO Running postdeploy actions for Cisco IOL 'R3' node
16:18:21 INFO Created link: R1:eth1 (Ethernet0/1) ▪┄┄▪ demolab_1:R1_Ethernet0/1
16:18:21 INFO Running postdeploy actions for Cisco IOL 'R2' node
16:18:22 INFO Created link: R2:eth1 (Ethernet0/1) ▪┄┄▪ demolab_1:R2_Ethernet0/1
16:18:22 INFO Created link: R3:eth1 (Ethernet0/1) ▪┄┄▪ demolab_1:R3_Ethernet0/1
16:18:22 INFO Adding host entries path=/etc/hosts
16:18:22 INFO Adding SSH config for nodes path=/etc/ssh/ssh_config.d/clab-demolab.conf
You are on the latest version (0.69.3)
╭─────────────────┬────────────────────────────┬─────────┬─────────────────╮
│       Name      │         Kind/Image         │  State  │  IPv4/6 Address │
├─────────────────┼────────────────────────────┼─────────┼─────────────────┤
│ clab-demolab-R1 │ cisco_iol                  │ running │ 192.168.121.101 │
│                 │ vrnetlab/cisco_iol:17.12.01 │         │ N/A             │
├─────────────────┼────────────────────────────┼─────────┼─────────────────┤
│ clab-demolab-R2 │ cisco_iol                  │ running │ 192.168.121.102 │
│                 │ vrnetlab/cisco_iol:17.12.01 │         │ N/A             │
├─────────────────┼────────────────────────────┼─────────┼─────────────────┤
│ clab-demolab-R3 │ cisco_iol                  │ running │ 192.168.121.103 │
│                 │ vrnetlab/cisco_iol:17.12.01 │        │ N/A            │
╰─────────────────┴────────────────────────────┴─────────┴─────────────────╯

┌──────────────────────────────────────────────────────────────────────────────────┐
│ DEPLOYING initial device configurations                                          │
└──────────────────────────────────────────────────────────────────────────────────┘
[WARNING]: Could not match supplied host pattern, ignoring: unprovisioned

PLAY [Deploy initial device configuration] ********************************************************************************************************
[WARNING]: Found variable using reserved name: hosts

TASK [Set variables that cannot be set with VARS] *************************************************************************************************
ok: [R3]
ok: [R2]
ok: [R1]

PLAY RECAP ****************************************************************************************************************************************
R1                         : ok=17   changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
R2                         : ok=16   changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
R3                         : ok=16   changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0

[SUCCESS] Lab devices configured
```

SSH into device is simple as below:

```terminal
❯ netlab connect R1
Connecting to clab-demolab-R1 using SSH port 22

R1#sh ip int bri
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            192.168.121.101 YES TFTP   up                    up
Ethernet0/1            172.16.0.1      YES manual up                    up
Ethernet0/2            unassigned      YES unset  administratively down down
Ethernet0/3            unassigned      YES unset  administratively down down
Loopback0              10.0.0.1        YES manual up                    up
R1#
```

You also don’t need to worry about interface names — the logic in **netlab** will automatically assign the next available interface.

```terminal
❯ netlab down --cleanup


┌──────────────────────────────────────────────────────────────────────────────────┐
│ CHECKING virtualization provider installation                                    │
└──────────────────────────────────────────────────────────────────────────────────┘
[SUCCESS] clab installed and working correctly

┌──────────────────────────────────────────────────────────────────────────────────┐
│ STOPPING clab nodes                                                              │
└──────────────────────────────────────────────────────────────────────────────────┘
16:26:28 INFO Parsing & checking topology file=clab.yml
16:26:28 INFO Parsing & checking topology file=clab.yml
16:26:28 INFO Destroying lab name=demolab
16:26:30 INFO Removed container name=clab-demolab-R3
16:26:31 INFO Removed container name=clab-demolab-R1
16:26:31 INFO Removed container name=clab-demolab-R2
16:26:31 INFO Removing host entries path=/etc/hosts
16:26:31 INFO Removing SSH config path=/etc/ssh/ssh_config.d/clab-demolab.conf

┌──────────────────────────────────────────────────────────────────────────────────┐
│ CLEANUP configuration files                                                      │
└──────────────────────────────────────────────────────────────────────────────────┘
... removing clab.yml
... removing ansible.cfg
... removing hosts.yml
... removing directory tree group_vars
... removing directory tree host_vars
... removing netlab.snapshot.yml
```

If you don’t want to remove everything, avoid using the --cleanup flag, as it will also destroy all the files. The [netlab tutorials](https://netlab.tools/tutorials/) page has dozens of guides describing different features — pick one to get started.

> Some command outputs have been omitted for brevity.
{: .prompt-info }

## Building a Campus Demo Lab

Now that we have Netlab and Containerlab ready, let’s move beyond simple topologies and create something closer to a real-world enterprise network.

To demonstrate, I built a **multi-layer redundant campus lab** using Netlab. It includes:

* **Core Layer**: A router for upstream connectivity.
* **Distribution Layer**: Two distribution switches with VRRP for gateway redundancy and OSPF for routing.
* **Access Layer**: Two access switches trunked up to the distribution.
* **Hosts**: Four Linux hosts spread across VLAN 10 and VLAN 20.

This gives us a design that resembles many real production networks, but completely containerized and automated with Netlab.

### Topology File

Here’s the topology definition I used (`topology.yml`). Netlab will take care of building configs, assigning IP addresses, and wiring everything up:

```yaml
---
name: campus
provider: clab

defaults.devices:
  iol.clab.image: vrnetlab/cisco_iol:17.12.01
  ioll2.clab.image: vrnetlab/cisco_iol:l2-17.12.01
  linux.clab.image: alpine:latest

nodes:
  R1:
    device: iol
    module: [ospf]
  D1:
  D2:
  S1:
  S2:
  H1:
  H2:
  H3:
  H4:

vlans:
  VLAN_10:
    id: 10
    links: [H1-S1, H4-S2]
  VLAN_20:
    id: 20
    links: [H2-S2, H3-S1]

groups:
  DISTRIBUTION:
    members: [D1, D2]
    device: ioll2
    module: [vlan, ospf]
    config: [configs]
    vlan:
      mode: irb
  ACCESS:
    members: [S1, S2]
    device: ioll2
    module: [vlan]
    vlan:
      mode: bridge
  VLAN_10:
    members: [H1, H4]
    device: linux
    module: [routing]
    role: host
    routing.static:
      - ipv4: 0.0.0.0/0
        nexthop.ipv4: 172.16.0.254
  VLAN_20:
    members: [H2, H3]
    device: linux
    module: [routing]
    role: host
    routing.static:
      - ipv4: 0.0.0.0/0
        nexthop.ipv4: 172.16.1.254

links:
  - R1-D1
  - R1-D2

  - group: access_trunk
    vlan.trunk: [VLAN_10, VLAN_20]
    members: [D1-S1, D1-S2, D2-S1, D2-S2]
```

This YAML defines a small **campus network lab** using **netlab** and **Containerlab**. It includes a core router, two distribution switches, two access switches, and four Linux hosts placed across two VLANs. Let’s go through the file step by step.

---

#### 🔹 Lab Header

```yaml
name: campus
provider: clab
```

* `name: campus` → The lab is called *campus*. This name is used for directories and container names.
* `provider: clab` → We’re deploying it on **Containerlab**, so netlab generates a `.clab.yml` file.

---

#### 🔹 Default Device Images

```yaml
defaults.devices:
  iol.clab.image: vrnetlab/cisco_iol:17.12.01
  ioll2.clab.image: vrnetlab/cisco_iol:l2-17.12.01
  linux.clab.image: alpine:latest
```

* Defines the **default Docker images** for each device type:

  * `iol` → Cisco IOS (used for routers).
  * `ioll2` → Cisco IOS L2 image (used for switches).
  * `linux` → Alpine Linux image (used for hosts).
* This way, you don’t need to repeat image info under each node.

---

#### 🔹 Nodes (Devices)

```yaml
nodes:
  R1:
    device: iol
    module: [ospf]
  D1:
  D2:
  S1:
  S2:
  H1:
  H2:
  H3:
  H4:
```

* `R1`: Core router using Cisco IOS. OSPF is enabled.
* `D1`, `D2`: Distribution switches (details will come from groups).
* `S1`, `S2`: Access switches.
* `H1–H4`: Linux hosts.

By leaving most nodes blank here, we let **groups** assign common properties later.

---

#### 🔹 VLAN Definitions

```yaml
vlans:
  VLAN_10:
    id: 10
    links: [H1-S1, H4-S2]
  VLAN_20:
    id: 20
    links: [H2-S2, H3-S1]
```

* `VLAN_10` (ID 10) connects host **H1** (via S1) and **H4** (via S2).
* `VLAN_20` (ID 20) connects host **H2** (via S2) and **H3** (via S1).

This creates **end-to-end VLANs across the two access switches**.

---

#### 🔹 Groups (Role-Based Configs)

Groups let us **assign common attributes** to multiple nodes.

##### Distribution Switches

```yaml
  DISTRIBUTION:
    members: [D1, D2]
    device: ioll2
    module: [vlan, ospf]
    config: [configs]
    vlan:
      mode: irb
```

* Members: `D1`, `D2`.
* Device type: `ioll2` (Cisco IOS L2).
* Modules: VLAN + OSPF.
* `config: [configs]` → Pulls extra config snippets from the `configs/` directory (your custom templates).
* `vlan.mode: irb` → Switch Virtual Interfaces (SVIs) are enabled for routing between VLANs.

##### Access Switches

```yaml
  ACCESS:
    members: [S1, S2]
    device: ioll2
    module: [vlan]
    vlan:
      mode: bridge
```

* Members: `S1`, `S2`.
* VLAN mode: *bridge only* (no routing). They act as pure Layer 2 switches.

##### VLAN_10 Hosts

```yaml
  VLAN_10:
    members: [H1, H4]
    device: linux
    module: [routing]
    role: host
    routing.static:
      - ipv4: 0.0.0.0/0
        nexthop.ipv4: 172.16.0.254
```

* Hosts: `H1` and `H4`.
* Device type: `linux`.
* Module: `routing` (so we can add static routes).
* Role: `host`.
* Static route: Default route points to `172.16.0.254` (gateway SVI on distribution switch).

##### VLAN_20 Hosts

```yaml
  VLAN_20:
    members: [H2, H3]
    device: linux
    module: [routing]
    role: host
    routing.static:
      - ipv4: 0.0.0.0/0
        nexthop.ipv4: 172.16.1.254
```

* Hosts: `H2` and `H3`.
* Default route via `172.16.1.254`.

This ensures **each host has Internet/default connectivity via its VLAN gateway**.

---

#### 🔹 Links (Physical Connections)

```yaml
links:
  - R1-D1
  - R1-D2
```

* Core router R1 connects directly to both distribution switches (D1 and D2).

```yaml
  - group: access_trunk
    vlan.trunk: [VLAN_10, VLAN_20]
    members: [D1-S1, D1-S2, D2-S1, D2-S2]
```

* Defines a trunk link group between distribution and access switches.
* Carries both VLAN\_10 and VLAN\_20.
* This builds the **classic campus topology**:

  * Core <-> Distribution <-> Access <-> Hosts.

---

### 🚀 What Happens When You Deploy

1. **R1**: Runs OSPF, connected to both D1 and D2.
2. **D1/D2**: Distribution switches, do VLAN routing with IRB + OSPF.
3. **S1/S2**: Access switches, bridge VLAN\_10 and VLAN\_20 traffic.
4. **H1–H4**: Linux hosts, auto-assigned IPs in their VLANs, each with a default route toward their VLAN gateway.

Once the file is ready, just bring up the lab with:

```bash
netlab up
```

Netlab will automatically generate configuration, and merged configs for the distribution switches (using `D1.cfg` and `D2.cfg` templates from `configs/`) and set up hosts with the right IP addresses.

```ruby
# D1.cfg Configuration
interface Vlan10
 vrrp 10 ip 172.16.0.254
 vrrp 10 priority 110
 vrrp 10 preempt

interface Vlan20
 vrrp 20 ip 172.16.1.254
 vrrp 20 priority 100
 vrrp 20 preempt

# D2.cfg Configuration
interface Vlan10
 vrrp 10 ip 172.16.0.254
 vrrp 10 priority 100
 vrrp 10 preempt

interface Vlan20
 vrrp 20 ip 172.16.1.254
 vrrp 20 priority 110
 vrrp 20 preempt
```

You can jump (`SSH`) into devices with:

```bash
netlab connect D1
netlab connect R1
```

---

### Verifying the Lab

* **VRRP** → D1 should be Master for VLAN 10, D2 for VLAN 20.
* **Host Connectivity** → H1 ↔ H4 (VLAN 10) and H2 ↔ H3 (VLAN 20) should ping.
* **Routing** → H1 should be able to ping H2 across VLANs.
* **OSPF** → R1, D1, D2 should form OSPF adjacencies.

That’s where Netlab comes in. Instead of manually dragging devices or repeating configs, you can define your entire lab in a simple YAML file. Netlab automatically provisions devices, assigns IPs, builds topologies, and pushes configs, saving huge amounts of time.

Whether you’re a student, a professional, or just curious about automation-driven labs, Netlab makes it easy to build and share realistic topologies in minutes.
