---
title:  "Getting Started with APIs for Network Automation"
date:   2025-08-08 13:00:00 +0500
categories:
  - python
  - networking
tags:
  - python3
  - cisco
  - csr1000v
  - restconf
  - network APIs
  - containerlab
  - network automation
  - network devices
  - network protocols
  - network management
  - network monitoring
---

## Understanding APIs in Networking
Modern network operations demand speed, consistency, and scalability. APIs (Application Programming Interfaces) have become a cornerstone for achieving this. Instead of manually logging into devices and running commands line by line, APIs let you send structured requests directly from your automation tools or scripts. The result: faster execution, fewer mistakes, and an environment that’s ready for large-scale orchestration.

This guide walks through APIs from a network engineer’s perspective — what they are, the types you’ll encounter, and how to start using them in real automation workflows. By the end, you’ll be ready to set up your own API-driven processes in a lab or production environment.

## What Are APIs?

At their core, APIs are agreements between systems. One system defines *how* it can be spoken to — the format of the requests, the commands it will accept, and the way it delivers responses. The other system follows those rules to exchange information or trigger actions.

In networking, an API is your bridge between automation logic and device control. If you want the operational status of all switches in your core, an API can deliver it in a single request, returning the data in machine-friendly formats like JSON or XML. This eliminates the need to parse CLI outputs manually and enables your scripts to take immediate action.

## Common API Styles in Networking

Different network APIs serve different purposes. Choosing the right one depends on your environment, the data exchange model, and the speed or efficiency you need.

* **REST (Representational State Transfer)** – Uses HTTP verbs like **GET**, **POST**, **PUT**, and **DELETE**. Data is often exchanged in JSON or XML. It’s simple, well-documented, and widely supported by network vendors.
* **gRPC (Google Remote Procedure Call)** – Built for performance. Runs over HTTP/2, supports streaming, and is often used for fast, repetitive interactions between systems or microservices.
* **WebSockets** – Maintains a persistent, bidirectional connection between client and server, enabling real-time event notifications or live telemetry streams.
* **Other styles** – GraphQL and SOAP appear in some environments, but REST and gRPC dominate most modern automation setups.

---

## How APIs Work in Network Automation

Think of an API call like asking a very precise question:

1. **Define what you want** – e.g., “Provide the operational state of interface Gig0/1.”
2. **Choose the method** – GET for retrieval, POST for creating new data, PUT for updates, DELETE for removal.
3. **Send it to the endpoint** – The device or controller’s API address that understands your request.

The response is structured and predictable, making it easy for your code to use it directly — whether to update a dashboard, feed into a monitoring system, or trigger corrective actions.

---

## Practical Use Cases for Network APIs

APIs allow engineers to go beyond manual device management. Here are common scenarios where they bring tangible benefits:

1. **Inventory Discovery** – Collect an up-to-date list of all devices, along with software versions and status, in seconds.
2. **Performance Monitoring** – Pull CPU, memory, and interface statistics without traditional SNMP polling delays.
3. **Configuration Deployment** – Apply changes such as VLAN updates or routing protocol adjustments across multiple devices simultaneously.
4. **Task Automation** – Schedule or trigger updates to ACLs, interface descriptions, or user accounts without manual intervention.
5. **System Integration** – Connect your network to external tools — when link failures occur.

## Tools for Working with APIs

* **Python `requests`** – The most accessible way to script API calls.
* **Postman** – Ideal for exploring and validating API endpoints before writing automation code.
* **cURL** – Fast, command-line-based API testing.

---

## Real-World Workflow Example

A straightforward process for using APIs in networking looks like this:

1. **Check API support** – Identify if the device or controller supports RESTCONF, gNMI, NETCONF, or vendor-specific APIs.
2. **Enable access** – Many systems require you to activate the API service in configuration.
3. **Test endpoints** – Use Postman or `curl` to verify authentication, connectivity, and expected responses.
4. **Script automation** – Write Python code to send requests, process responses, and optionally log activity.
5. **Integrate and scale** – Incorporate the scripts into Ansible, Nornir, or CI/CD pipelines for repeatable execution.

## Testing a Network Device API in the Lab

For this example, we’ll use a **Cisco CSR1000v** running inside Containerlab. This will help you practice basic API requests before moving on to larger automation workflows.

**Lab Details:**

* **Device:** CSR1000v (Containerlab)
* **IP Address:** `172.20.20.2`
* **Username:** `admin`
* **Password:** `admin`
* **API Protocol:** RESTCONF over HTTPS

### 1. Check API Availability with cURL

We’ll query the device’s `ietf-interfaces` data model to confirm RESTCONF access.

```bash
curl -k --user 'admin:admin' --request GET 'https: //172.20.20.2:443/restconf/data/ietf-interfaces:interfaces' --header 'Accept: application/yang-data+json'
```

**What happens here:**

* `-k` bypasses SSL verification (useful for lab tests)
* `-u admin:admin` sends the credentials
* The URL points to the RESTCONF endpoint for interface data

If successful, you should see a JSON list of interfaces:

```
{
    "ietf-interfaces:interfaces": {
        "interface": [
            {
                "name": "GigabitEthernet1",
                "description": "Containerlab management interface",
                "type": "iana-if-type:ethernetCsmacd",
                "enabled": true,
                "ietf-ip:ipv4": {
                    "address": [
                        {
                            "ip": "10.0.0.15",
                            "netmask": "255.255.255.0"
                        }
                    ]
                },
                "ietf-ip:ipv6": {
                    "address": [
                        {
                            "ip": "2001:db8::2",
                            "prefix-length": 64
                        }
                    ]
                }
            },
            {
                "name": "GigabitEthernet2",
                "type": "iana-if-type:ethernetCsmacd",
                "enabled": false,
                "ietf-ip:ipv4": {},
                "ietf-ip:ipv6": {}
            }
        ]
    }
}
```

### 2. Fetch Data with Python

Here’s a simple Python script using the requests library:

```python
from pprint import pprint

import requests
from requests.auth import HTTPBasicAuth

url = "https://172.20.20.2/restconf/data/ietf-interfaces:interfaces"

headers = {"Accept": "application/yang-data+json"}

# Disable warnings for self-signed certs
requests.packages.urllib3.disable_warnings()

response = requests.get(
    url, headers=headers, auth=HTTPBasicAuth("admin", "admin"), verify=False
)

if response.status_code == 200:
    # print("Interfaces Data:")
    pprint(response.json().get("ietf-interfaces:interfaces"))
else:
    print(f"Request failed with status code {response.status_code}")
```

```
{'interface': [{'description': 'Containerlab management interface',
                'enabled': True,
                'ietf-ip:ipv4': {'address': [{'ip': '10.0.0.15',
                                              'netmask': '255.255.255.0'}]},
                'ietf-ip:ipv6': {'address': [{'ip': '2001:db8::2',
                                              'prefix-length': 64}]},
                'name': 'GigabitEthernet1',
                'type': 'iana-if-type:ethernetCsmacd'},
               {'enabled': False,
                'ietf-ip:ipv4': {},
                'ietf-ip:ipv6': {},
                'name': 'GigabitEthernet2',
                'type': 'iana-if-type:ethernetCsmacd'}]}
```

Python makes it easy to automate these API calls. Using the `requests` library.

### 3. Configure an Interface via RESTCONF

Now let’s configure a loopback interface using RESTCONF. This example shows how to build a JSON payload and send it to the device.

```python
import json

import requests
from requests.auth import HTTPBasicAuth
from urllib3.exceptions import InsecureRequestWarning

# Your simple inputs
ip_address = "172.16.1.100"
netmask = "255.255.255.0"
interface = "Loopback0"

# Device details
device = {
    "host": "172.20.20.2",  # CSR1000v RESTCONF IP
    "port": 443,
    "username": "admin",
    "password": "admin",
}

# Build JSON payload automatically
payload = {
    "ietf-interfaces:interface": {
        "name": interface,
        "description": "Configured by RESTCONF",
        "type": "iana-if-type:softwareLoopback",
        "enabled": True,
        "ietf-ip:ipv4": {"address": [{"ip": ip_address, "netmask": netmask}]},
    }
}

# Disable SSL warnings (lab use only)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

# Send RESTCONF request
url = f"https://{device['host']}:{device['port']}/restconf/data/ietf-interfaces:interfaces/interface={interface}"
headers = {
    "Content-Type": "application/yang-data+json",
    "Accept": "application/yang-data+json",
}

response = requests.put(
    url,
    data=json.dumps(payload),
    auth=HTTPBasicAuth(device["username"], device["password"]),
    headers=headers,
    verify=False,
)

# Print result
if response.status_code in [200, 201, 204]:
    print(f"✅ Interface {interface} configured successfully.")
else:
    print(f"❌ Failed with status code {response.status_code}")
    print(response.text)
```

```
✅ Interface Loopback0 configured successfully.
```

This script builds a JSON payload for a loopback interface and sends it to the CSR1000v device. On success, it will create the interface with the specified IP address and netmask.

## Best Practices for API-Based Automation

* Always use **HTTPS** to secure traffic.
* Implement **error handling** for timeouts, invalid responses, or authentication failures.
* **Log every action** to assist with troubleshooting.
* Keep changes **small and atomic** to reduce the blast radius of mistakes.
* Prefer **tokens or keys** over plain-text passwords.

## Security Considerations

* Enforce authentication and authorization for every call.
* Validate all incoming and outgoing data.
* Keep your API clients and libraries updated to avoid security flaws.

## Conclusion

APIs let network engineers replace manual, error-prone tasks with fast, consistent automation. With the right tools and a safe lab setup, you can quickly learn to query, configure, and integrate devices into broader workflows.

The formula is simple: **learn the API, test it, automate it, and embed it into your processes**. Once in place, your network operations become faster, more reliable, and easier to manage.
