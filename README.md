# Cisco Routing Commands Cheat Sheet

A concise and organized reference of Cisco routing configuration and verification commands. This cheat sheet is ideal for CCNA/CCNP students, lab enthusiasts, and networking professionals.

![types-of-dynamic-routing-protocol-1024x494](https://github.com/user-attachments/assets/4dae71f0-9c77-4f36-9b8f-346f75cb88ec)

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
7. [Route Redistribution](#7-route-redistribution)
8. [Policy-Based Routing (PBR)](#8-policy-based-routing-pbr)
9. [Route Filtering Techniques](#9-route-filtering-techniques)
10. [Resources & Contributions](#10-resources--contributions)

---

## 1. Static Routing

```bash
ip route <destination IP > <mask> <next-hop | exit-interface>  # Basic static route
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

```bash
router ospf 1
 router-id 1.1.1.1                        # Set router ID
 network 192.168.1.0 0.0.0.255 area 0     # Define area 0
 passive-interface default
 no passive-interface GigabitEthernet0/0
```

#### Multi-Area OSPF

```bash
network 10.1.1.0 0.0.0.255 area 1
network 10.2.2.0 0.0.0.255 area 2
```

#### OSPF Authentication

```bash
interface GigabitEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 CISCO123
router ospf 1
 area 0 authentication message-digest
```

#### OSPF Summarization (ABR)

```bash
area 1 range 192.168.0.0 255.255.0.0
```

#### OSPF Stub & NSSA

```bash
router ospf 1
 area 1 stub
 area 2 nssa
```

#### Virtual Link (for disconnected areas)

```bash
area 1 virtual-link 2.2.2.2
```

#### OSPFv3 (IPv6)

```bash
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
router ospf 1 ipv6
 router-id 1.1.1.1
```

---

### 3.2. Exterior Gateway Protocols (EGPs)

Used between different autonomous systems (ASes).

#### Path Vector Protocol

- **BGP (Border Gateway Protocol)** – The only widely-used EGP
  - **BGP-4** – Current standard
  - **MP-BGP** – Supports multiple address families (IPv4/IPv6/MPLS)

> 🔎 We will focus on the most widely used one: **BGP**.

### ✅ BGP Full Configuration Guide

```bash
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 192.0.2.2 remote-as 65002
 network 203.0.113.0 mask 255.255.255.0
```

#### iBGP Mesh

```bash
neighbor 10.1.1.1 remote-as 65001
neighbor 10.1.1.2 remote-as 65001
```

#### BGP Authentication

```bash
neighbor 192.0.2.2 password mypassword
```

#### BGP Timers

```bash
neighbor 192.0.2.2 timers 10 30
```

#### Prefix Advertise Control

```bash
ip prefix-list MY_LIST seq 5 permit 203.0.113.0/24
router bgp 65001
 neighbor 192.0.2.2 prefix-list MY_LIST out
```

#### Route Reflectors

```bash
router bgp 65001
 bgp cluster-id 1.1.1.1
 neighbor 10.1.1.2 route-reflector-client
```

#### Confederation

```bash
router bgp 65001
 bgp confederation identifier 65000
 bgp confederation peers 65010 65020
```

#### Soft Reconfiguration

```bash
neighbor 192.0.2.2 soft-reconfiguration inbound
clear ip bgp * soft
```

#### BGP IPv6

```bash
address-family ipv6
 neighbor 2001:db8::2 activate
 neighbor 2001:db8::2 route-map IPV6-IN in
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
| EIGRP (internal) | 90       |
| OSPF             | 110      |
| RIP              | 120      |
| EIGRP (external) | 170      |
| Unknown          | 255      |

---

## 6. Route Manipulation & Useful Routing Commands

```bash
ip route <dest> <mask> <next-hop> <AD>   # Modify admin distance
ip summary-address eigrp <asn> <network> <mask>  # Manual summarization
ip route <network> <mask> null0         # Discard route
```

---

## 7. Route Redistribution

Redistributing routes between different routing protocols.

### Basic Redistribution (EIGRP to OSPF example):

```bash
router ospf 1
 redistribute eigrp 100 subnets
```

### Mutual Redistribution:

```bash
router ospf 1
 redistribute eigrp 100 subnets
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
```

### With Route-Map Filtering:

```bash
route-map REDIST_OSPF_TO_EIGRP permit 10
 match ip address 10
 set metric 10000 100 255 1 1500
router eigrp 100
 redistribute ospf 1 route-map REDIST_OSPF_TO_EIGRP
```

---

## 8. Policy-Based Routing (PBR)

PBR allows routing decisions based on policies rather than the routing table.

```bash
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
route-map PBR_POLICY permit 10
 match ip address 100
 set ip next-hop 10.0.0.1
interface GigabitEthernet0/0
 ip policy route-map PBR_POLICY
```

---

## 9. Route Filtering Techniques

### Distribute-List Filtering (OSPF Example)

```bash
access-list 1 deny 192.168.1.0 0.0.0.255
access-list 1 permit any
router ospf 1
 distribute-list 1 out GigabitEthernet0/0
```

### Prefix List Filtering (BGP Example)

```bash
ip prefix-list BLOCK_ROUTES seq 5 deny 10.0.0.0/8
ip prefix-list BLOCK_ROUTES seq 10 permit 0.0.0.0/0 le 32
router bgp 65001
 neighbor 203.0.113.2 prefix-list BLOCK_ROUTES out
```

### Route-Map Filtering with Tags

```bash
route-map FILTER_TAG deny 10
 match tag 10
route-map FILTER_TAG permit 20
router ospf 1
 distribute-list route-map FILTER_TAG in
```

---

## 10. Resources & Contributions

📬 Contributions, corrections, and improvements are welcome. Open a pull request or create an issue!

---

**Made with 💻 by Nidhal**\
[LinkedIn](https://www.linkedin.com/in/nidhal-labri/)

