# 04 — OSPF Multi-Router Network

## What I Built
A 4-router OSPF network in a ring topology, with point-to-point links between each router and a loopback interface on each representing a simulated network. This lab demonstrates single-area OSPF configuration, neighbor adjacency formation under two different OSPF network types, equal-cost path selection on a ring, and full failure/recovery behavior when a link goes down.

## Topology
![Topology](04-ospf-multirouter.png)

| Router | Loopback | Connected Links |
|---|---|---|
| R1 | 1.1.1.1/32 | R1–R2 (10.0.0.0/30), R1–R4 (10.0.0.12/30) |
| R2 | 2.2.2.2/32 | R1–R2 (10.0.0.0/30), R2–R3 (10.0.0.4/30) |
| R3 | 3.3.3.3/32 | R2–R3 (10.0.0.4/30), R3–R4 (10.0.0.8/30) |
| R4 | 4.4.4.4/32 | R3–R4 (10.0.0.8/30), R1–R4 (10.0.0.12/30) |

Each router connects to two neighbors, forming a ring: R1 — R2 — R3 — R4 — R1. This gives every router two equal-cost paths to reach the router directly opposite it, and ensures the network has a redundant path available before any link failure testing.

## IP Addressing Scheme

| Link / Loopback | Network | Router A | Router B |
|---|---|---|---|
| R1–R2 | 10.0.0.0/30 | R1: 10.0.0.1 | R2: 10.0.0.2 |
| R2–R3 | 10.0.0.4/30 | R2: 10.0.0.5 | R3: 10.0.0.6 |
| R3–R4 | 10.0.0.8/30 | R3: 10.0.0.9 | R4: 10.0.0.10 |
| R4–R1 | 10.0.0.12/30 | R4: 10.0.0.13 | R1: 10.0.0.14 |
| R1 Loopback | 1.1.1.1/32 | — | — |
| R2 Loopback | 2.2.2.2/32 | — | — |
| R3 Loopback | 3.3.3.3/32 | — | — |
| R4 Loopback | 4.4.4.4/32 | — | — |

Each point-to-point link uses its own dedicated /30 subnet rather than sharing a single large block, giving each link its own broadcast domain — standard practice for router-to-router links.

## Key Configuration

**OSPF process (example: R1):**
```
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.12 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
```

**Interface addressing (example: R1):**
```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
!
interface GigabitEthernet0/1
 ip address 10.0.0.14 255.255.255.252
```

Each router's OSPF process is locally significant — `router ospf 1` does not need to match between routers, only the **area number** does. Full configs for all four routers are in [`configs/`](./configs).

## OSPF Network Type: Broadcast vs. Point-to-Point

By default, OSPF treats Ethernet interfaces as network type **broadcast**, which triggers DR/BDR election even on a link with only two routers. This was the initial state of the lab:

```
Neighbor ID     Pri   State        Address       Interface
4.4.4.4         1     FULL/BDR     10.0.0.13     GigabitEthernet0/1
2.2.2.2         1     FULL/BDR     10.0.0.2      GigabitEthernet0/0
```

Since these are physically point-to-point links, the network type was changed on every router-facing interface to reflect that:

```
interface GigabitEthernet0/0
 ip ospf network point-to-point
```

After the change, DR/BDR election no longer occurs (priority drops to 0, role shows as `-`):

```
Neighbor ID     Pri   State      Address       Interface
4.4.4.4         0     FULL/ -    10.0.0.13     GigabitEthernet0/1
2.2.2.2         0     FULL/ -    10.0.0.2      GigabitEthernet0/0
```

Route costs and path selection were unaffected by this change — only the adjacency model changed. All further testing (verification and failure/recovery) was done using point-to-point network type.

## Verification

**Full neighbor adjacency — all four routers, point-to-point:**

| Router | Neighbors |
|---|---|
| R1 | 2.2.2.2 (FULL/-), 4.4.4.4 (FULL/-) |
| R2 | 1.1.1.1 (FULL/-), 3.3.3.3 (FULL/-) |
| R3 | 2.2.2.2 (FULL/-), 4.4.4.4 (FULL/-) |
| R4 | 1.1.1.1 (FULL/-), 3.3.3.3 (FULL/-) |

**Routing table — R1 (`show ip route ospf`), fully converged:**
```
O   2.2.2.2/32 [110/2] via 10.0.0.2, GigabitEthernet0/0
O   3.3.3.3/32 [110/3] via 10.0.0.2, GigabitEthernet0/0
              [110/3] via 10.0.0.13, GigabitEthernet0/1
O   4.4.4.4/32 [110/2] via 10.0.0.13, GigabitEthernet0/1
O   10.0.0.4/30  [110/2] via 10.0.0.2, GigabitEthernet0/0
O   10.0.0.8/30  [110/2] via 10.0.0.13, GigabitEthernet0/1
```

R1 reaches 3.3.3.3 (the router directly opposite it on the ring) via **two equal-cost paths** — through R2 and through R4 — both at cost 3. This is expected OSPF behavior (ECMP) on a symmetric ring topology, and it's the property that makes the network resilient to a single link failure.

All four routers were confirmed to have full visibility of every other router's loopback and every link subnet in the topology.

## Failure & Recovery Test

**Scenario:** Simulated a link failure between R1 and R2 by shutting down R1's GigabitEthernet0/0 interface, then observed how the network rerouted around it.

**Before failure (R1):**
```
O   2.2.2.2/32 [110/2] via 10.0.0.2, GigabitEthernet0/0
O   3.3.3.3/32 [110/3] via 10.0.0.2, GigabitEthernet0/0
              [110/3] via 10.0.0.13, GigabitEthernet0/1
```

**Failure triggered:**
```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#shutdown
```

**Immediately after — neighbor and route tables (R1):**
```
Neighbor ID     State      Address       Interface
4.4.4.4         FULL/ -    10.0.0.13     GigabitEthernet0/1
```
The 2.2.2.2 neighbor adjacency is gone entirely (the link itself is down, not just a route change).

```
O   2.2.2.2/32 [110/4] via 10.0.0.13, GigabitEthernet0/1
O   3.3.3.3/32 [110/3] via 10.0.0.13, GigabitEthernet0/1
O   4.4.4.4/32 [110/2] via 10.0.0.13, GigabitEthernet0/1
```
Traffic to 2.2.2.2 rerouted around the ring through R4 → R3 → R2. Cost increased from 2 to 4, reflecting the longer path. The route to 3.3.3.3 dropped from two equal-cost paths to a single path (through R4), since the R2-side path no longer exists. No network became unreachable — the ring's redundancy absorbed the failure. Reconvergence completed in well under 15 seconds, with the new route showing an install time of just a few seconds after the interface was shut down.

**Recovery:**
```
R1(config-if)#no shutdown
```

Caught mid-recovery, the neighbor table briefly showed the adjacency rebuilding:
```
Neighbor ID     State      Address       Interface
4.4.4.4         FULL/ -    10.0.0.13     GigabitEthernet0/1
2.2.2.2         INIT/ -    10.0.0.2      GigabitEthernet0/0
```
followed by a syslog message confirming the adjacency completing its state machine transition:
```
%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/0 from LOADING
```

**Fully recovered (R1):**
```
O   2.2.2.2/32 [110/2] via 10.0.0.2, GigabitEthernet0/0
O   3.3.3.3/32 [110/3] via 10.0.0.13, GigabitEthernet0/1
              [110/3] via 10.0.0.2, GigabitEthernet0/0
O   4.4.4.4/32 [110/2] via 10.0.0.13, GigabitEthernet0/1
```
The network returned to its original state — direct path to 2.2.2.2 restored, dual equal-cost paths to 3.3.3.3 restored. Recovery completed in roughly the same timeframe as the failure, with new routes showing install times of only a few seconds.

**Key takeaway:** OSPF's link-state model means a single point-to-point link failure doesn't require waiting on a dead-timer expiration to be detected — the interface going down is detected immediately, triggering an immediate LSA flood and SPF recalculation. On a ring topology with equal-cost alternate paths, this means reconvergence is fast and no destination becomes unreachable, since every router already has a backup path precomputed in its topology database.

---

## Files in This Directory
| File | Description |
|---|---|
| README.md | This documentation file |
| 04-ospf-multirouter.png | Network topology diagram |
| 04-ospf-multirouter.pkt | Cisco Packet Tracer lab file |
| R1_running-config.txt | Router 1 running configuration |
| R2_running-config.txt | Router 2 running configuration |
| R3_running-config.txt | Router 3 running configuration |
| R4_running-config.txt | Router 4 running configuration |

---

## Tools Used
- Cisco Packet Tracer 8.x
- GitHub for version control and documentation

## Previous Project
[Project 03 — Inter-VLAN Routing](../../tier-01-foundation/03-intervlan-routing/) — Router-on-a-stick configuration that this project's routing concepts build on.

## Next Project
Coming soon — Project 05: STP & Redundant Switching Lab
