# Project 01 — Basic LAN: Dual Subnet Routed Network

## Overview
A routed dual-subnet LAN simulating a small office environment 
with two departments — Sales and IT. Built in Cisco Packet Tracer 
using one 4331 router, two 2960 switches, and eight end hosts.

**Topology:** 1 Router | 2 Switches | 8 PCs | 2 Subnets

![Network Topology](topology.png)

---

## Network Design
Two /27 subnets carved from 192.168.1.0/24 — one per department.

| Subnet | Network | Mask | Gateway | Host Range |
|---|---|---|---|---|
| Sales | 192.168.1.0/27 | 255.255.255.224 | 192.168.1.1 | .2 — .30 |
| IT | 192.168.1.32/27 | 255.255.255.224 | 192.168.1.33 | .34 — .62 |

---

## IP Addressing Scheme

| Device | Interface | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0/0 | 192.168.1.1 | 255.255.255.224 | — |
| R1 | G0/0/1 | 192.168.1.33 | 255.255.255.224 | — |
| PC0 (Sales) | NIC | 192.168.1.2 | 255.255.255.224 | 192.168.1.1 |
| PC1 (Sales) | NIC | 192.168.1.3 | 255.255.255.224 | 192.168.1.1 |
| PC2 (Sales) | NIC | 192.168.1.4 | 255.255.255.224 | 192.168.1.1 |
| PC3 (Sales) | NIC | 192.168.1.5 | 255.255.255.224 | 192.168.1.1 |
| PC4 (IT) | NIC | 192.168.1.34 | 255.255.255.224 | 192.168.1.33 |
| PC5 (IT) | NIC | 192.168.1.35 | 255.255.255.224 | 192.168.1.33 |
| PC6 (IT) | NIC | 192.168.1.36 | 255.255.255.224 | 192.168.1.33 |
| PC7 (IT) | NIC | 192.168.1.37 | 255.255.255.224 | 192.168.1.33 |

---

## Key Configuration

### Router R1 — Interface Configuration

enable

configure terminal
interface g0/0/0

ip address 192.168.1.1 255.255.255.224

no shutdown
interface g0/0/1

ip address 192.168.1.33 255.255.255.224

no shutdown

### Why no shutdown is required
Cisco router interfaces are administratively down by default. 
Unlike switch ports which come up automatically, router interfaces 
must be explicitly enabled with no shutdown or they will not pass 
traffic regardless of whether a cable is connected.

---

## Verification

### show ip interface brief output
Both interfaces confirmed up/up after configuration:
- G0/0/0 — 192.168.1.1 — up/up
- G0/0/1 — 192.168.1.33 — up/up

### Ping Test Results
| Source | Destination | Result |
|---|---|---|
| PC0 (192.168.1.2) | PC1 (192.168.1.3) | ✅ Success |
| PC0 (192.168.1.2) | R1 G0/0/0 (192.168.1.1) | ✅ Success |
| PC0 (192.168.1.2) | R1 G0/0/1 (192.168.1.33) | ✅ Success |
| PC0 (192.168.1.2) | PC4 (192.168.1.34) | ✅ Success |
| PC4 (192.168.1.34) | PC7 (192.168.1.37) | ✅ Success |

All inter-subnet pings successful — routing confirmed working.

---

## What I Learned
- How to design a subnetting scheme using /27 from a /24 block
- Block size calculation — 256 minus last octet of mask = 32 for /27
- How router interfaces connect two separate subnets and act as 
  the default gateway for each subnet
- Why subnet masks must be consistent across all devices in a subnet
- How to verify interface status using show ip interface brief
- The difference between /27 (30 hosts) and /26 (62 hosts) and 
  when to choose each based on host requirements
- Why router interfaces require no shutdown unlike switch ports

---

## Troubleshooting Encountered
During initial configuration G0/0/1 showed up/down status. 
Identified the cable between R1 and SW2 was connected to the 
wrong port on the router. Reconnected to the correct interface 
and confirmed up/up status with show ip interface brief.

---

## Files in This Directory
| File | Description |
|---|---|
| README.md | This documentation file |
| topology.png | Network topology diagram |
| 01-basic-lan.pkt | Cisco Packet Tracer lab file |
| R1-config.txt | Router R1 running configuration |

---

## Tools Used
- Cisco Packet Tracer 8.x
- GitHub for version control and documentation
