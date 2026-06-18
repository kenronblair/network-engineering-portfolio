## Verification & Testing

### show vlan brief output confirmed:
- VLAN 10 SALES — active on Fa0/2, Fa0/3 (SW1) and Fa0/2, Fa0/3 (SW2)
- VLAN 20 IT — active on Fa0/4, Fa0/5 (SW1) and Fa0/4, Fa0/5 (SW2)
- VLAN 30 MANAGEMENT — active on Fa0/1 (SW3)

### show interfaces trunk confirmed:
- Gig0/1 on SW1, SW2, SW3 — trunking, 802.1Q, VLANs 1,10,20,30 active

### Ping Test Results

| Source | Destination | VLAN Relationship | Result |
|---|---|---|---|
| PC0 VLAN10 SW1 | PC1 VLAN10 SW1 | Same VLAN same switch | ✅ Success |
| PC0 VLAN10 SW1 | PC4 VLAN10 SW2 | Same VLAN different switch | ✅ Success |
| PC2 VLAN20 SW1 | PC6 VLAN20 SW2 | Same VLAN different switch | ✅ Success |
| PC0 VLAN10 SW1 | PC2 VLAN20 SW1 | Different VLAN same switch | ❌ Fail |
| PC0 VLAN10 SW1 | PC6 VLAN20 SW2 | Different VLAN different switch | ❌ Fail |
| PC0 VLAN10 SW1 | PC8 VLAN30 SW3 | Different VLAN different switch | ❌ Fail |
| PC8 VLAN30 SW3 | PC0 VLAN10 SW1 | Different VLAN | ❌ Fail |

---

## Why Inter-VLAN Pings Fail
VLANs are separate Layer 2 broadcast domains. Traffic cannot cross 
from one VLAN to another without a Layer 3 routing decision. No 
router or Layer 3 switch is configured for inter-VLAN routing in 
this lab — that is implemented in Project 03. The failed pings are 
not a misconfiguration — they prove the VLANs are working correctly.

---

## What I Learned
- VLANs logically segment a network at Layer 2 regardless of 
  physical switch location — a Sales PC on SW1 and a Sales PC on 
  SW2 are in the same broadcast domain through trunk links
- Trunk ports carry traffic for multiple VLANs simultaneously 
  using 802.1Q encapsulation — access ports carry traffic for 
  one VLAN only
- VLANs must be created on each switch independently — they do 
  not automatically propagate without VTP
- Inter-VLAN communication requires a Layer 3 device — a router 
  or Layer 3 switch — to make routing decisions between VLANs
- The management VLAN should use a completely separate subnet 
  from data VLANs to ensure proper segmentation
- show vlan brief and show interfaces trunk are the primary 
  verification commands for VLAN and trunk configuration

---

## Troubleshooting Encountered
Initially assigned all four SW1 PCs to VLAN 10 using interface 
range. Used show vlan brief to identify the error and corrected 
by reassigning Fa0/4 and Fa0/5 to VLAN 20 using switchport 
access vlan 20. This reinforced the importance of verifying 
port assignments immediately after configuration.

---

## Files in This Directory
| File | Description |
|---|---|
| README.md | This documentation file |
| 02-vlan-segmentation.png | Network topology diagram |
| 02-vlan-segmentation.pkt | Cisco Packet Tracer lab file |
| SW1_running-config.txt | Switch 1 running configuration |
| SW2_running-config.txt | Switch 2 running configuration |
| SW3_running-config.txt | Switch 3 running configuration |

---

## Tools Used
- Cisco Packet Tracer 8.x
- GitHub for version control and documentation

## Next Project
[Project 03 — Inter-VLAN Routing](../03-intervlan-routing/) — 
Adding a router-on-a-stick configuration to enable routing 
between VLANs 10, 20, and 30.
