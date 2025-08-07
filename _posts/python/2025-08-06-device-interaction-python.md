---
title:  "Exploring Network Device Interactions with Python"
date:   2025-08-06 15:14:00 +0500
categories:
  - python
  - networking
tags:
  - python3 automation
  - Network configuration
  - network automation
  - paramiko
  - telnetlib
  - pysnmp
  - network devices
  - network protocols
  - SSH
  - Telnet
  - SNMP
---

In network engineering, the ability to interact with network devices programmatically is invaluable. Python, with its simplicity and powerful libraries, is an excellent tool for automating and managing network tasks. This post will guide you through using Python for low-level network device interactions using protocols like SSH, Telnet, and SNMP.

## [The Importance of Network Automation]({% post_url networking/2023-08-29-network-automation %})

### Why Automate?

Network automation is about using software to perform repetitive network management tasks. This not only saves time but also cuts down on human error, leading to more reliable and consistent network operations.

### The Role of Python

Python stands out in the world of network automation due to its readability and wide library support. Whether you're configuring devices, monitoring network performance, or troubleshooting issues, Python provides the tools you need to get the job done efficiently.

## Using Telnet for Network Device Interactions

### Introduction to Telnet

Telnet is an older protocol used for remote access to network devices. Unlike SSH, Telnet does not provide encrypted communication, which makes it less secure. Still, it's used in some older systems.

### Python Libraries for Telnet

#### Telnetlib

Telnetlib is a Python module that provides a Telnet client class. It's part of the Python standard library, making it easy to use right away.

##### [Basic Usage]({% post_url python/2021-11-29-telnetlib %})

Here's a simple example of how to use Telnetlib to connect to a network device and execute a command:

```python
import telnetlib

# Connect to the device
tn = telnetlib.Telnet('192.168.1.1')

# Login
tn.read_until(b"Username: ")
tn.write(b"admin\n")
tn.read_until(b"Password: ")
tn.write(b"securepassword\n")

# Execute a command
tn.write(b"show version\n")
print(tn.read_all().decode())

# Close the connection
tn.close()
```

##### [More Usage](https://github.com/sydasif/network-automation-script)

Telnetlib can also be used for tasks like handling timeouts and errors:

```python
import telnetlib

# Connect to the device with a timeout
tn = telnetlib.Telnet('192.168.1.1', timeout=10)

try:
    # Login
    tn.read_until(b"Username: ")
    tn.write(b"admin\n")
    tn.read_until(b"Password: ")
    tn.write(b"securepassword\n")

    # Execute a command
    tn.write(b"show version\n")
    print(tn.read_very_eager().decode())

except Exception as e:
    print(f"An error occurred: {e}")

finally:
    # Close the connection
    tn.close()
```

## Using SSH for Network Device Interactions

### Introduction to SSH

Secure Shell (SSH) is a protocol used to securely access network devices. It provides encrypted communication, ensuring that sensitive data stays protected during transmission.

### Python Libraries for SSH

#### Paramiko

Paramiko is a Python library that implements the SSHv2 protocol, providing both client and server functionality. It's a popular choice for SSH interactions in Python due to its ease of use and wide feature set.

##### [Basic Usage]({% post_url python/2021-12-14-paramiko %})

Here's a simple example of how to use Paramiko to connect to a network device and execute a command:

```python
import paramiko

# Create an SSH client
ssh_client = paramiko.SSHClient()
ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# Connect to the device
ssh_client.connect('192.168.1.1', username='admin', password='securepassword')

# Execute a command
stdin, stdout, stderr = ssh_client.exec_command('show version')
print(stdout.read().decode())

# Close the connection
ssh_client.close()
```

##### [More Usage](https://github.com/sydasif/network-automation-script)

Paramiko can also be used for tasks like transferring files using SFTP (SSH File Transfer Protocol):

```python
import paramiko

# Create an SSH client
ssh_client = paramiko.SSHClient()
ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh_client.connect('192.168.1.1', username='admin', password='securepassword')

# Open an SFTP session
sftp = ssh_client.open_sftp()

# Transfer a file from the local machine to the remote device
sftp.put('local_file.txt', 'remote_file.txt')

# Transfer a file from the remote device to the local machine
sftp.get('remote_file.txt', 'local_file.txt')

# Close the SFTP session and the SSH connection
sftp.close()
ssh_client.close()
```

## Using SNMP for Network Device Interactions

Simple Network Management Protocol (SNMP) is a protocol used for gathering and organizing information about managed devices on IP networks. It's widely used for monitoring and control.

### [Python Libraries for SNMP](https://docs.lextudio.com/pysnmp/v7.1/)

PySNMP is a Python library that provides a pure-Python SNMP engine. It supports SNMP v1, v2c, and v3, making it a flexible tool for SNMP-based tasks.

Current PySNMP stable version is 7.1. It runs with Python 3.9+ and is recommended for new applications as well as for migration from older, now obsolete, PySNMP releases.

## Lab Setup

The lab consists of:

- A Cisco IOL router (IP: `192.168.122.10`)
- SNMP enabled with community string `public`
- Python environment with `pysnmp` installed

```bash
pip install pysnmp
```

> If you are using Containerlab, make sure SNMP is enabled in the Cisco IOL node startup config.

### Sample SNMP Config for Cisco IOL

```text
snmp-server community public RO
snmp-server location Lab
snmp-server contact neteng@example.com
```

## Query sysDescr Using SNMP v2c (AsyncIO)

Here’s a Python script that uses asyncio with `pysnmp` to retrieve the device’s system description:

```python
import asyncio

from pysnmp.hlapi.v3arch.asyncio import (
    CommunityData,
    ContextData,
    ObjectIdentity,
    ObjectType,
    SnmpEngine,
    UdpTransportTarget,
    get_cmd,
)


async def run():
    snmp_engine = SnmpEngine()

    # 👇 Replace with your Cisco IOL device IP
    transport = await UdpTransportTarget.create(("192.168.122.10", 161))

    result = await get_cmd(
        snmp_engine,
        CommunityData("public", mpModel=1),  # SNMP v2c
        transport,
        ContextData(),
        ObjectType(ObjectIdentity("SNMPv2-MIB", "sysDescr", 0)),
    )

    error_indication, error_status, error_index, var_binds = result

    if error_indication:
        print(f"[!] Error: {error_indication}")
    elif error_status:
        print(f"[!] SNMP Error: {error_status.prettyPrint()} at index {error_index}")
    else:
        for var_bind in var_binds:
            print(" = ".join([x.prettyPrint() for x in var_bind]))

    snmp_engine.close_dispatcher()


if __name__ == "__main__":
    asyncio.run(run())
```

## Output Example

```
SNMPv2-MIB::sysDescr.0 = Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX-ADVENTERPRISEK9-M), Version 17.12.1, RELEASE SOFTWARE (fc5)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2023 by Cisco Systems, Inc.
Compiled Thu 27-Jul-23 22:33 by m
```

## Query Multiple SNMP OIDs

This one queries `sysDescr`, `sysName`, and `sysUpTime` from the same device:

```python
import asyncio

from pysnmp.hlapi.v3arch.asyncio import (
    CommunityData,
    ContextData,
    ObjectIdentity,
    ObjectType,
    SnmpEngine,
    UdpTransportTarget,
    get_cmd,
)


async def run():
    snmp_engine = SnmpEngine()
    ip = "192.168.122.10"
    transport = await UdpTransportTarget.create((ip, 161))

    oids = [
        ObjectType(ObjectIdentity("SNMPv2-MIB", "sysDescr", 0)),
        ObjectType(ObjectIdentity("SNMPv2-MIB", "sysName", 0)),
        ObjectType(ObjectIdentity("SNMPv2-MIB", "sysUpTime", 0)),
    ]

    result = await get_cmd(
        snmp_engine, CommunityData("public", mpModel=1), transport, ContextData(), *oids
    )

    error_indication, error_status, error_index, var_binds = result

    if error_indication:
        print(f"[!] Error: {error_indication}")
    elif error_status:
        print(f"[!] SNMP Error: {error_status.prettyPrint()} at index {error_index}")
    else:
        print(f"SNMP Data for {ip}:")
        for var_bind in var_binds:
            print(" = ".join([x.prettyPrint() for x in var_bind]))

    snmp_engine.close_dispatcher()


if __name__ == "__main__":
    asyncio.run(run())
```

## Output Example

```
SNMP Data for 192.168.122.10:
SNMPv2-MIB::sysDescr.0 = Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX-ADVENTERPRISEK9-M), Version 17.12.1, RELEASE SOFTWARE (fc5)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2023 by Cisco Systems, Inc.
Compiled Thu 27-Jul-23 22:33 by m
SNMPv2-MIB::sysName.0 = RTR.lab
SNMPv2-MIB::sysUpTime.0 = 621498
```

You can extend this script to:
  - Pull interface stats
  - Monitor uptime
  - Trigger events based on SNMP data
  - Feed the data into monitoring tools or your Django API


## Best Practices and Security Considerations

### Best Practices

* **Use Secure Protocols**: Prefer SSH over Telnet for connecting to network devices.
* **Error Handling**: Write proper error-handling code to manage unexpected problems and keep your scripts stable.
* **Logging**: Keep logs for tracking and troubleshooting. Logs help track changes and find issues faster.

### Security Considerations

* **Credential Management**: Use environment variables or secret management tools to store passwords. Avoid putting them in your scripts.
* **Access Control**: Limit who can access network devices and who can run your scripts.
* **Regular Updates**: Keep your code and libraries updated to avoid known problems or security flaws.

## Conclusion

Python is a strong tool for network engineers, letting them automate tasks, monitor network performance, and manage device settings. By working with protocols like SSH, Telnet, and SNMP, you can improve how you manage your network and help make it more reliable and secure. Learn how to build these tools and improve your daily workflow with practical scripts.
