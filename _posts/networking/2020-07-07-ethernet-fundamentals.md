---
title:  "Ethernet Fundamentals"
date:   2020-07-07 15:02:15 +0500
categories: networking
tags:
  - ethernet fundamentals
  - networking
---

## Overview of LANs
The term Ethernet refers to a family of LAN standards that define physical and data-link layers of the world’s most popular wired LAN technology. The cabling, the connectors of the cables, the protocol rules, to create an Ethernet LAN.

## Ethernet Physical Layer Standards

Ethernet supports a large variety of options for physical links, includes many standards for different kinds of optical and copper cabling, and speeds as far as the types and length of the cables.

UTP cabling transmits data over electrical circuits via the copper wires inside the cable. Fiber-optic cabling, allows data to send, over a light in glass fiber in the center of the cable. Optical cables allow longer cabling distances between nodes.

## Cables Types of Ethernet

| Speed     | Name             | Standard Name | IEEE Standard | Type & length   |
| --------- | ---------------- | ------------- | ------------- | --------------- |
| 10 Mbps   | Ethernet         | 10Base-T      | 802.3         | Copper, 100 Mtr |
| 100 Mbps  | Fast Ethernet    | 100Base-T     | 802.3u        | Copper, 100 Mtr |
| 1000 Mbps | Gigabit Ethernet | 1000Base-LX   | 802.3z        | Fiber, 5000 Mtr |
| 1000 Mbps | Gigabit Ethernet | 1000Base-T    | 802.3ab       | Copper, 100 Mtr |
| 10 Gbps   | 10 Gig Ethernet  | 10GBase-T     | 802.3an       | Copper, 100 Mtr |

## Ethernet Data-Link Layer

Ethernet uses the same data-link layer standard over all types of physical links. That standard defines a common Ethernet header and trailer, over a UTP or any kind of fiber cable, regardless of speed, the data-link header and trailer use the same format.

While the physical layer standards emphasize sending bits in form of electrical or light signals over a cable.

Ethernet LAN is a combination of user devices, LAN switches, and different kinds of cabling. Each link can use different types of cables, at different speeds.

They all work together to deliver Ethernet frames from the one device on the LAN to some other device.
