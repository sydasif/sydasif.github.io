---
title:  "Introduction to TCP/IP Networking"
date:   2020-07-06 15:02:15 +0500
categories: networking
tags:
  - tcp-ip
  - networking
---

## TCP/IP Networking Model
A set of protocols, physical requirements, and logical rules that devices must follow to communicate with other network devices.

The TCP/IP model also avoids repeating work already done by some other standards body. As an example, the Institute of Electrical and Electronic Engineers (IEEE) defines Ethernet LANs; the TCP/IP model refers to IEEE Ethernet as an option.

For example, voltage and current levels used on a particular cable (UTP, optical fiber, or wireless) when transmitting data. To understand a TCP/IP model breaks the functions into a small number of categories called layers. Each layer includes its own protocols and standards related to that category.

TCP/IP model layers are below:-

1. Application
2. Transport
3. Network
4. Data Link
5. Physical layer

The TCP/IP model defines common terms and layers used. The bottom layer focuses on how to transmit bits over each individual link.

The data-link layer focuses on sending data over one type of physical link: for instance, networks use different data-link protocols for Ethernet LANs versus wireless LANs.

The network layer focuses on delivering data over the entire path from the original sending computer to the final destination computer.

And the top two layers focus more on the applications that need to send and receive data.

| TCP/IP Layer         | Protocol                 |
| -------------------- | ------------------------ |
| Application          | HTTP, POP3, SMTP         |
| Transport            | TCP, UDP                 |
| Internet             | IP, ICMP                 |
| Data Link & Physical | Ethernet, 802.11 (Wi-Fi) |
| -------------------- | ------------------------ |

## Encapsulation of Data

A five-step process by which a TCP/IP host sends data as mention below:-

1. Create and encapsulate the application data with any required application layer headers, for example, HTTP get message in the header.

2. Encapsulate the data providing by the application layer inside a transport-layer header, a TCP or UDP header is used for end-user applications.

3. Encapsulate the data providing by the transport layer inside a network layer (IP) header. IP defines the IP addresses that uniquely identify each computer.

4. Encapsulate the data given by the network layer inside a data-link layer header and trailer. This layer uses both a header and a trailer.

5. Transmit the bits. The physical layer encodes a signal onto the medium to transmit the frame.

![pic](/assets/img/Data_encap.PNG)

In networking, terms are used like segment, packet, and frame. Each term has a particular meaning, referring to the headers (trailers) defined by a particular layer and the data encapsulated following by that header.

Segment for the transport layer, packet for the network layer, and frame for the link layer.
