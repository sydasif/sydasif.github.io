---
title:  "Getting Started with Scrapli - Python Network Automation"
date:   2025-09-17 13:00:00 +0500
categories:
  - python
  - networking
tags:
  - python3
  - cisco
  - containerlab
  - network automation
  - network devices
  - network protocols
  - network management
  - network monitoring
---

In today post, I introduce an old but lesser known Python library named scrapli, to automate network devices. There are so many module/libraries in Python for network engineers to automate networking tasks. So the question is why scrapli vs netmiko. According to ChatGPT:

>Netmiko is like the veteran — reliable, proven, and widely used for network automation over SSH. Scrapli came later as a modern alternative, built to be faster, more flexible, and developer-friendly. It handles both synchronous and asynchronous operations, works smoothly with structured data, and is easier to extend for different platforms, which makes it attractive if you want speed and scalability without giving up reliability.
{: .prompt-info }

## Scrapli - Introduction

According to the [Scrapli](https://carlmontanari.github.io/scrapli/) docs, it is a Python library for connecting to network devices like routers, switches, and firewalls over SSH or Telnet. The name comes from “scrape CLI.” Its goal is speed, flexibility, and simplicity, with a clean API that works in both sync and async modes, backed by solid testing and documentation.

### Setup Scrapli and Devices

In most cases, installing via pip is the easiest and recommended way to install Scrapli. It’s best to use a virtual environment to avoid system issues or conflicts with other modules.

```bash
$ pip install scrapli

Collecting scrapli
  Downloading scrapli-2025.1.30-py3-none-any.whl.metadata (11 kB)
Downloading scrapli-2025.1.30-py3-none-any.whl (145 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 145.7/145.7 kB 704.5 kB/s eta 0:00:00
Installing collected packages: scrapli
Successfully installed scrapli-2025.1.30
```

The next step is testing Scrapli with network devices. To get your feet wet with automation, the easiest way is to use **[netlab.tools](https://netlab.tools/)** along with **Containerlab**.

Here’s a minimal topology file:

```yml
---
name: lab
provider: clab

defaults:
  devices:
    iol.clab.image: vrnetlab/cisco_iol:17.12.01
    ioll2.clab.image: vrnetlab/cisco_iol:l2-17.12.01

nodes:
  R1:
    device: iol
  S1:
    device: ioll2

links:
  - R1-S1
```

👉 For lab setup guides, see this [netlab tutorial](https://python.plainenglish.io/netlab-tutorial-build-a-campus-network-lab-vlans-vrrp-and-ospf-e52a9b8631b1) and [Containerlab walkthrough](https://medium.com/@sydasif78/how-containerlab-transformed-my-network-lab-setup-2b64b97113a1).

### A Simple Example

Here’s a short Python script using Scrapli to connect to a Cisco device and run a command:

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "192.168.121.101",
    "auth_username": "admin",
    "auth_password": "admin",
    "auth_strict_key": False,
}

conn = IOSXEDriver(**device)
conn.open()
response = conn.send_command("show ip interface brief")
print(response.result)
conn.close()
```

Upon running the script, we get the following output:

```bash
$ python show_cmd.py
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            192.168.121.101 YES TFTP   up                    up
Ethernet0/1            10.1.0.1        YES manual up                    up
Ethernet0/2            unassigned      YES unset  administratively down down
Ethernet0/3            unassigned      YES unset  administratively down down
Loopback0              10.0.0.1        YES manual up                    up
```

This confirms that Scrapli successfully connected to the Cisco device and retrieved the CLI output of the `show ip interface brief` command.

### Using `with` Statement for Cleaner Connections

The previous example works, but it requires us to manually open and close the connection. A cleaner way is to use a `with` statement, which automatically handles connection setup and teardown.

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "192.168.121.101",
    "auth_username": "admin",
    "auth_password": "admin",
    "auth_strict_key": False,
}

with IOSXEDriver(**device) as conn:
    response = conn.send_command("show ip interface brief")
print(response.result)
```

This is better than the previous example because the connection is automatically closed when the `with` block finishes, making the code safer and easier to manage.

You can check the available methods of Scrapli objects in a few simple ways using *IPython*:

```py
In [1]: from scrapli.driver.core import IOSXEDriver
   ...:
   ...: device = {
   ...:     "host": "192.168.121.101",
   ...:     "auth_username": "admin",
   ...:     "auth_password": "admin",
   ...:     "auth_strict_key": False,
   ...: }
   ...:
   ...: conn = IOSXEDriver(**device)

In [2]: print(dir(conn))

['acquire_priv', 'auth_bypass', 'auth_password', 'auth_private_key', 'auth_private_key_passphrase', 'auth_secondary', 'auth_strict_key', 'auth_username', 'channel', 'close', 'commandeer', 'comms_prompt_pattern', 'comms_prompt_search_depth', 'comms_return_char', 'comms_roughly_match_inputs', 'default_desired_privilege_level', 'failed_when_contains', 'genie_platform', 'get_prompt', 'host', 'isalive', 'logger', 'on_close', 'on_init', 'on_open', 'open', 'port', 'privilege_levels', 'read_callback', 'send_and_read', 'send_command', 'send_commands', 'send_commands_from_file', 'send_config', 'send_configs', 'send_configs_from_file', 'send_interactive', 'ssh_config_file', 'ssh_known_hosts_file', 'textfsm_platform', 'timeout_ops', 'timeout_socket', 'timeout_transport', 'transport', 'transport_name', 'update_privilege_levels']
```

This will list all methods and attributes for the connection object.

### [TextFSM/NTC-Templates Integration](https://carlmontanari.github.io/scrapli/user_guide/basic_usage/#textfsmntc-templates-integration)

Scrapli supports parsing CLI output into structured data using **TextFSM** and **ntc-templates**. This requires installing the TextFSM extra:

```bash
# for zsh
$ pip install "scrapli[textfsm]"

# for bash
$ pip install scrapli[textfsm]
```

If TextFSM parsing succeeds, Scrapli returns a structured result (list of dictionaries). If parsing fails, it simply returns an empty list.

### Example

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "192.168.121.101",
    "auth_username": "admin",
    "auth_password": "admin",
    "auth_strict_key": False,
}

with IOSXEDriver(**device) as conn:
    response = conn.send_command("show ip interface brief")
    structured_result = response.textfsm_parse_output()

print(structured_result)
```

### Output

```bash
$ python show_cmd.py
[{'interface': 'Ethernet0/0', 'ip_address': '192.168.121.101', 'status': 'up', 'proto': 'up'}, {'interface': 'Ethernet0/1', 'ip_address': '10.1.0.1', 'status': 'up', 'proto': 'up'}, {'interface': 'Ethernet0/2', 'ip_address': 'unassigned', 'status': 'administratively down', 'proto': 'down'}, {'interface': 'Ethernet0/3', 'ip_address': 'unassigned', 'status': 'administratively down', 'proto': 'down'}, {'interface': 'Loopback0', 'ip_address': '10.0.0.1', 'status': 'up', 'proto': 'up'}]
```

As you can see, instead of raw CLI output, Scrapli + TextFSM gives you structured JSON-like data that’s much easier to use in automation scripts.

### Pretty JSON Output

For easier reading, you can format the structured result with Python’s `json` module:

```python
import json
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "192.168.121.101",
    "auth_username": "admin",
    "auth_password": "admin",
    "auth_strict_key": False,
}

with IOSXEDriver(**device) as conn:
    response = conn.send_command("show ip interface brief")
    structured_result = response.textfsm_parse_output()

print(json.dumps(structured_result, indent=2))
```

### Output

```bash
$ python show_cmd.py
[
  {
    "interface": "Ethernet0/0",
    "ip_address": "192.168.121.101",
    "status": "up",
    "proto": "up"
  },
  {
    "interface": "Ethernet0/1",
    "ip_address": "10.1.0.1",
    "status": "up",
    "proto": "up"
  },
  {
    "interface": "Ethernet0/2",
    "ip_address": "unassigned",
    "status": "administratively down",
    "proto": "down"
  },
  {
    "interface": "Ethernet0/3",
    "ip_address": "unassigned",
    "status": "administratively down",
    "proto": "down"
  },
  {
    "interface": "Loopback0",
    "ip_address": "10.0.0.1",
    "status": "up",
    "proto": "up"
  }
]
```

This makes the output much easier to read, especially when working with larger command results.

Let’s expand the example to handle **two devices** — one at `192.168.121.101` and another at `192.168.121.102`. We’ll loop through both, run the command, and print results.

### Multiple Devices Example

```python
import json
from scrapli.driver.core import IOSXEDriver

devices = [
    {
        "host": "192.168.121.101",
        "auth_username": "admin",
        "auth_password": "admin",
        "auth_strict_key": False,
    },
    {
        "host": "192.168.121.102",
        "auth_username": "admin",
        "auth_password": "admin",
        "auth_strict_key": False,
    },
]

for device in devices:
    with IOSXEDriver(**device) as conn:
        response = conn.send_command("show ip interface brief")
        structured_result = response.textfsm_parse_output()
    print(f"\n=== Results from {device['host']} ===")
    print(json.dumps(structured_result, indent=2))
```

### Sample Output

```bash
=== Results from 192.168.121.101 ===
[
  {
    "interface": "Ethernet0/0",
    "ip_address": "192.168.121.101",
    "status": "up",
    "proto": "up"
  },
  {
    "interface": "Ethernet0/1",
    "ip_address": "10.1.0.1",
    "status": "up",
    "proto": "up"
  },
  ...
]

=== Results from 192.168.121.102 ===
[
  {
    "interface": "Ethernet0/0",
    "ip_address": "192.168.121.102",
    "status": "up",
    "proto": "up"
  },
  {
    "interface": "Ethernet0/1",
    "ip_address": "10.2.0.1",
    "status": "up",
    "proto": "up"
  },
  ...
]
```

This way, you can scale the same script to handle many devices just by adding them to the `devices` list.

### [Sending Configurations](https://carlmontanari.github.io/scrapli/user_guide/basic_usage/#sending-configurations)

Scrapli makes sending configs simple. You can push a single block with `send_config`, multiple lines with `send_configs`, or load them directly from a file using `send_configs_from_file`. Each option automatically handles privilege escalation, so you can focus on the commands instead of the access level.

```py
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "192.168.121.101",
    "auth_username": "admin",
    "auth_password": "admin",
    "auth_strict_key": False,
}

with IOSXEDriver(**device) as conn:
    response = conn.send_configs([
        "interface loopback123",
        "description configured by scrapli",
    ])
print(response.result)
```

### Output

```bash
$ python config_cmd.py
interface loopback123
description configured by scrapli
```

Scrapli doesn’t exit config mode after sending configs. That’s intentional — the network drivers automatically return to the correct privilege level before running any new command, so you don’t need to manually exit config mode.

## Conclusion

Scrapli is a modern, fast, and flexible alternative to Netmiko that makes network automation smoother. It works well for both commands and configurations, integrates with TextFSM for structured output, and scales easily across multiple devices. With its clean API, async support, and built-in handling of privilege levels, it’s a solid tool for Python network automation.
