---
layout: post
title: "Juniper Notes 2 - Tech"
date: 2018-05-10
categories: [Reflection]
---

**Note: Donyel's areas... Juniper’s WAN SDN Controller, NorthStar Controller**

- Router: a networking device that forwards data packets between computer networks (mediate data in a computer network) by processing the routing information included in the packet or datagram
- Computer Network/Data Network: Allows the exchange of data amongst computing devices using nodes (data links), established over cable media or wireless media
- Network Nodes: network computer devices that originate, route, terminate data (PCs, phones, servers, networking hardware)
  - In most cases, application-specific communications protocol are layered
- Communications Protocol: system of rules that allows two entities of a communications system to transmit information (can be developed into a technical standard)
  - Group of protocols designed to work together known as *protocol suites*, upon software implementation known as a *protocol stack*
  - Often specified w/ algorithms and data structures :)
  - Interfaced with a machine's operating system framework, often the TCP/IP model and OSI model frameworks
  - Protocols must address:
    - Data formats for data exchange
    - Address formats for data exchange
    - Mapping addresses between different schema
    - Routing, as part of internetworking
    - Detection of transmission errors
    - Acknowledgements
    - Timeouts and retries
    - Direction of information flow
    - Sequence control
    - Flow control (regulating speed, if transmitter and receiver have different capabilities)
  - TCP/IP Protocol Stack:
    - Application Layer
    - Transport Layer (Between Applications)
    - Internet Layer (Between Machines)
    - Link Layer (Communicating with the local network link)
  - Program translation layered as well:
    - Compiler (programming language 'translator'), assembler (creates object code), link editor (combine several object files into a single executable), loader (places programs in memory, prepares them for execution)
  - Often standardized by the ISO
- Network Packet: a formatted unit of data (list of bits/bytes) carried by a packet-switched network
  - Contain control information and user data (payload)
- Network Interface Controller (NIC): computer hardware that provides a computer with the ability to access transmission media
- Repeater: receives a network signal, cleans it of unnecessary noise, regenerates/retransmits it
- Ethernet hub: repeater with multiple ports
- Bridges: connects and filters traffic between two network segments at the data link layer to form a single network
  - Local bridges: directly connect LANs
  - Remote bridges: creates a wide area network link between LANs; often replaced with routers
  - Wireless bridges: joins LANs or connects remote devices
- **Network switches: forwards and filters OSI layer 2 datagrams (frames) between ports based on the destination MAC address in each frame**
- Modems: connects network nodes via wire not originally designated for digital network traffic, or for wireless
- **Firewall: network device for controlling network security and access rules**
- Network Topologies
  - !['Tops'](https://upload.wikimedia.org/wikipedia/commons/thumb/9/97/NetworkTopologies.svg/220px-NetworkTopologies.svg.png)
  - 
