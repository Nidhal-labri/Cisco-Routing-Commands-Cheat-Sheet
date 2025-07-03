# Cisco Routing Commands Cheat Sheet

A concise and organized reference of Cisco routing configuration and verification commands. This cheat sheet is ideal for CCNA/CCNP students, lab enthusiasts, and networking professionals.

![types-of-dynamic-routing-protocol-1024x494](https://github.com/user-attachments/assets/b1eefa9c-a33f-44d3-9ab3-8f4b2883b451)

---

## 📌 Table of Contents

1. [Static Routing](#1-static-routing)
2. [Default Routing](#2-default-routing)
3. [Dynamic Routing Protocols](#3-dynamic-routing-protocols)
   - [Interior Gateway Protocols (IGPs)](#interior-gateway-protocols-igps)
     - [Distance Vector Protocols](#distance-vector-protocols)
     - [Link State Protocols](#link-state-protocols)
   - [Exterior Gateway Protocols (EGPs)](#exterior-gateway-protocols-egps)
4. [Routing Verification Commands](#4-verification-commands)
5. [Administrative Distance](#5-administrative-distance)
6. [Route Manipulation & Useful Routing Commands](#6-route-manipulation--useful-routing-commands)

---

## 1. Static Routing

```bash
ip route <destination IP > <mask> <next-hop | exit-interface>  # Basic static route
ip route 0.0.0.0 0.0.0.0 <next-hop>                            # Default static route
ip route <dest> <mask> null0                                   # Blackhole route (drop traffic)
```

---

## 2. Default Routing

```bash
ip route 0.0.0.0 0.0.0.0 <next-hop>       # Send all unmatched traffic to next hop
```

---

## 3. Dynamic Routing Protocols

Dynamic routing protocols enable routers to exchange routing information automatically. These protocols are divided into IGPs (used within an autonomous system) and EGPs (used between ASes).

### 3.1. Interior Gateway Protocols (IGPs)

#### 3.1.1. Distance Vector Protocols

These protocols calculate the best path based on distance (hops) and direction. Many exist, but some are obsolete:

- **RIP (Routing Information Protocol)** – Classic distance-vector protocol *(obsolete)*
  - **RIPv2** – Supports classless routing, multicast updates
  - **RIPng** – RIP for IPv6
- **IGRP** – Cisco proprietary *(obsolete)*
- **EIGRP** – Cisco proprietary (hybrid protocol, classified as distance-vector)

#### 3.1.2. Link State Protocols

Protocols that build a complete map of the network using LSAs:

- **OSPF (Open Shortest Path First)** – Most commonly used IGP
  - **OSPFv3** – OSPF version 3 for IPv6
- **IS-IS (Intermediate System to Intermediate System)** – Common in service provider networks

> 🔎 We will focus on the most widely used one: **OSPF**.

### ✅ OSPF Full Configuration Guide

**Cost Calculation:**

Cost = Reference Bandwidth / Interface Bandwidth
Default Reference Bandwidth = 100 Mbps


| Attribute      | Value                    |
| -------------- | ------------------------ |
| Type           | Link-State               |
| Algorithm      | Dijkstra (SPF)           |
| Metric         | Cost (Bandwidth-based)   |
| AD             | 110                      |
| RFC            | 2328 (IPv4), 5340 (IPv6) |
| Protocol       | IP                       |
| Transport      | IP protocol 89           |
| Authentication | Plaintext, MD5           |
| AllSPF Routers | 224.0.0.5                |
| AllDR Routers  | 224.0.0.6                |

#### OSPF Packet Types

| Type  | Description                      |
| ----- | -------------------------------- |
| Hello | Discover and maintain neighbors  |
| DBD   | Summary of LSDB for comparison   |
| LSR   | Request specific LSAs            |
| LSU   | Send LSAs (in reply or flooding) |
| LSAck | Acknowledge receipt of LSAs      |

#### Router Types

| Router Type     | Description                                         |
| --------------- | --------------------------------------------------- |
| Internal Router | All interfaces in one area                          |
| Backbone Router | Has at least one interface in area 0 (the backbone) |
| ABR             | Connects multiple OSPF areas                        |
| ASBR            | Connects to another routing domain (e.g., BGP)      |

#### OSPF Tables

| Table Type     | Command                 |
| -------------- | ----------------------- |
| Neighbor Table | `show ip ospf neighbor` |
| Topology Table | `show ip ospf database` |
| Routing Table  | `show ip route ospf`    |


#### Example 1: Single-Area OSPF (With Optional Security)
![image](https://github.com/user-attachments/assets/2493ddfc-aa91-4ad3-acc2-6d9298563caf)

```bash
# Base Configuration for R1
interface loopback0  
 ip address 10.1.1.1 255.255.255.255
interface serial0/0
 ip address 192.168.12.1 255.255.255.0
 ip ospf authentication message-digest        # (optional) Enable MD5 auth on the interface
 ip ospf message-digest-key 1 md5 SecurePass123  # (optional) Define MD5 key

router ospf 100
 router-id 1.1.1.1
 area 0 authentication message-digest         # (optional) Enable auth in area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 10.1.1.1 0.0.0.0 area 0

# Base Configuration for R2
interface loopback0
 ip address 10.2.2.2 255.255.255.255
interface serial0/0
 ip address 192.168.12.2 255.255.255.0
 ip ospf authentication message-digest        # (optional) Enable MD5 auth
 ip ospf message-digest-key 1 md5 SecurePass123  # (optional) Define key

router ospf 100
 router-id 2.2.2.2
 area 0 authentication message-digest         # (optional) Match area auth
 network 192.168.12.0 0.0.0.255 area 0
 network 10.2.2.2 0.0.0.0 area 0
```

#### Example 2: Multi-Area with ABR + Summarization + NSSA + Virtual-Link (Optional Enhancements)
![image](https://github.com/user-attachments/assets/ceeb4dbc-cb0d-40ba-940e-454844f86f1b)

```bash
# Router R1 (ABR)
interface loopback0
 ip address 10.1.1.1 255.255.255.255
interface serial0/0
 ip address 192.168.12.1 255.255.255.0
interface serial0/1
 ip address 192.168.13.1 255.255.255.0

router ospf 100
 router-id 1.1.1.1
 network 192.168.12.1 0.0.0.0 area 0
 network 192.168.13.1 0.0.0.0 area 1
 network 10.1.1.1 0.0.0.0 area 0
 area 1 range 192.168.13.0 255.255.255.0        # (optional) Summarization
 area 1 nssa                                    # (optional) Make area 1 NSSA

# Router R2 (Backbone Router)
interface loopback0
 ip address 10.2.2.2 255.255.255.255
interface serial0/0
 ip address 192.168.12.2 255.255.255.0

router ospf 100
 router-id 2.2.2.2
 network 192.168.12.2 0.0.0.0 area 0
 network 10.2.2.2 0.0.0.0 area 0
 area 1 virtual-link 1.1.1.1                    # (optional) Virtual link to reach area 1

# Router R3 (Internal NSSA Router)
interface serial0/0
 ip address 192.168.13.3 255.255.255.0
interface loopback0
 ip address 10.3.3.3 255.255.255.255

router ospf 100
 router-id 3.3.3.3
 network 192.168.13.3 0.0.0.0 area 1
 network 10.3.3.3 0.0.0.0 area 1
 area 1 nssa                                    # (optional) Area 1 is NSSA
```

#### OSPF Troubleshooting Commands

```bash
show ip protocols
show ip ospf [process-id]
show ip ospf interface [brief | <interface>]
show ip ospf neighbor
show ip ospf database
debug ip ospf [hello | adjacency | events]
```
---

### 3.2. Exterior Gateway Protocols (EGPs)

#### Path Vector Protocol – BGP

- **BGP (Border Gateway Protocol)**
  - **BGP-4** – Current standard
  - **MP-BGP** – Multiprotocol BGP

#### About BGP

| Attribute      | Value          |
| -------------- | -------------- |
| Type           | Path Vector    |
| Algorithm      | Path Selection |
| eBGP AD        | 20             |
| iBGP AD        | 200            |
| RFC Standard   | RFC 4271       |
| Protocols      | IP             |
| Transport      | TCP port 179   |
| Authentication | MD5            |

#### Terminology

| Term                | Description                                                      |
| ------------------- | ---------------------------------------------------------------- |
| Autonomous System   | A domain managed by a single organization                        |
| External BGP (eBGP) | BGP peering between different ASes                               |
| Internal BGP (iBGP) | BGP peering within the same AS                                   |
| Synchronization     | IGP must know a route before BGP advertises it (legacy behavior) |

### ✅ BGP Full Configuration Example

![DD](https://github.com/user-attachments/assets/93389b28-29c5-4494-bac9-8f60e9bb9fd2)


**Router A**

```bash
interface Serial1/0
 description Backbone to B
 ip address 172.16.0.1 255.255.255.252
interface Serial1/1
 description Backbone to C
 ip address 172.16.0.5 255.255.255.252
interface FastEthernet2/0
 description LAN
 ip address 192.168.1.1 255.255.255.0

router bgp 65100
 no synchronization
 network 172.16.0.0 mask 255.255.255.252
 network 172.16.0.4 mask 255.255.255.252
 network 192.168.1.0
 neighbor South peer-group
 neighbor South remote-as 65200
 neighbor 172.16.0.2 peer-group South
 neighbor 172.16.0.6 peer-group South
 no auto-summary
```

**Router B**

```bash
interface FastEthernet0/0
 description Local to C
 ip address 10.0.0.1 255.255.255.252
interface Serial1/0
 description Backbone to A
 ip address 172.16.0.2 255.255.255.252
interface FastEthernet2/0
 description LAN
 ip address 192.168.2.1 255.255.255.0

router ospf 100
 network 10.0.0.1 0.0.0.0 area 0
 network 192.168.2.0 0.0.0.255 area 1

router bgp 65200
 no synchronization
 redistribute ospf 100 route-map LAN_Subnets
 neighbor 10.0.0.2 remote-as 65200
 neighbor 172.16.0.1 remote-as 65100
 no auto-summary

access-list 10 permit 192.168.0.0 0.0.255.255
route-map LAN_Subnets permit 10
 match ip address 10
 set metric 100
```

**Router C**

```bash
interface FastEthernet0/0
 description Local to B
 ip address 10.0.0.2 255.255.255.252
interface Serial1/0
 description Backbone to A
 ip address 172.16.0.6 255.255.255.252
interface FastEthernet2/0
 description LAN
 ip address 192.168.3.1 255.255.255.0

router ospf 100
 network 10.0.0.2 0.0.0.0 area 0
 network 192.168.3.0 0.0.0.255 area 2

router bgp 65200
 no synchronization
 redistribute ospf 100 route-map LAN_Subnets
 neighbor 10.0.0.1 remote-as 65200
 neighbor 172.16.0.5 remote-as 65100
 no auto-summary

access-list 10 permit 192.168.0.0 0.0.255.255
route-map LAN_Subnets permit 10
 match ip address 10
 set metric 100
```

#### BGP Troubleshooting

```bash
show ip bgp
show ip bgp summary
show ip bgp neighbors
show ip route [bgp]
clear ip bgp * [soft]
debug ip bgp events
debug ip bgp updates
```
---

## 4. Verification Commands

```bash
show ip route                            # View routing table
show ip protocols                        # View routing protocol settings
show ip ospf neighbor                    # OSPF neighbor status
show ip eigrp neighbors                  # EIGRP neighbors
show ip rip database                     # RIP routing info
show run | section router                # Show routing config block
```

---

## 5. Administrative Distance (AD)

| Protocol         | AD Value |
| ---------------- | -------- |
| Connected        | 0        |
| Static           | 1        |
| eBGP             | 20       |
| EIGRP (internal) | 90       |
| IGRP             | 100      |
| OSPF             | 110      |
| IS-IS            | 115      |
| RIP              | 120      |
| EIGRP (external) | 170      |
| iBGP             | 200      |
| Unknown          | 255      |

---

## 6. Route Manipulation & Useful Routing Commands

```bash
ip route <dest> <mask> <next-hop> <AD>   # Modify admin distance
ip summary-address eigrp <asn> <network> <mask>  # Manual summarization
ip route <network> <mask> null0         # Discard route
```

---

## 🧠 Contributions

Contributions, corrections, and improvements are welcome. Open a pull request or create an issue!

---

**Made with 💻 by Nidhal** | **[LinkedIn](https://www.linkedin.com/in/nidhal-labri/)**


