# 01 - Multi-Area OSPF Architecture & Partitioned Backbone Recovery

## 1. Architectural Overview

A single-area OSPF deployment does not scale in large ISP cores because every link flap forces all routers to execute the Dijkstra Shortest Path First (SPF) algorithm and flood Type 1 (Router) and Type 2 (Network) LSAs domain-wide.

This architecture segments the topology into four distinct failure domains:

- **Area 0 (Backbone Core):** Interconnects `R1`, `R2`, `R3`, and `R4` over a shared multi-access broadcast segment (`172.16.10.0/24`).
- **Area 51 (Standard NSSA):** Isolates `R2`, `R4`, and `R5`, blocking external Type 5 LSAs from entering while permitting external branch redistribution (`R6`) via Type 7 LSAs.
- **Area 2 (Transit Area):** Connects `R3` to `R7` (`192.168.3.0/24`) and carries an OSPF Virtual Link to stitch the partitioned Area 0 segment on `R8`.
- **Area 3 (NBMA WAN):** Connects `R3`, `R9`, and `R10` (`192.168.9.0/24`) using Point-to-Multipoint Non-Broadcast mode.

---

## 2. Interface Network Types & DR/BDR Optimization

- **Area 0 Broadcast (`172.16.10.0/24`):** Retains the default `BROADCAST` network type across `R1–R4` on `Ethernet0/0`, utilizing a Designated Router (DR) and Backup Designated Router (BDR) to reduce $O(n^2)$ adjacency meshes to $O(n)$.
- **Area 2 Transit Link (`R3` to `R7`):** Configured as `point-to-point` to bypass the 40-second DR/BDR `Wait` timer, accelerating link convergence and eliminating unnecessary Type 2 Network LSAs.
- **Host Subnet (`R7 Ethernet0/1` - `192.168.7.0/24`):** Configured as a `passive-interface` so the subnet is advertised in `R7`'s Type 1 LSA without transmitting OSPF Hello packets toward end-host `H1`.

```cisco
! R3 & R7 Point-to-Point Transit Optimization
interface Ethernet0/0
 ip ospf network point-to-point

! R7 Host Subnet Protection
router ospf 1
 passive-interface Ethernet0/1
```

3. Discontiguous Area 0 & Virtual Link Engineering
   R8 resides in Area 0 (172.16.20.0/24 and 8.8.8.8/32) but is physically separated from the primary Area 0 switch (SW1) by Area 2. Because OSPF loop-prevention rules dictate that ABRs only process Type 3 Summary LSAs learned via a direct Area 0 attachment, R7 cannot inject R8's prefixes into the rest of the network without a logical tunnel.

An OSPF Virtual Link (VL0) is established between R3 (RID 3.3.3.3) and R7 (RID 7.7.7.7) across transit Area 2:

Cisco CLI
! On R3
router ospf 1
area 2 virtual-link 7.7.7.7

! On R7
router ospf 1
area 2 virtual-link 3.3.3.3
Operational Mechanics of VL0
Unicast Tunneling: OSPF packets on VL0 are encapsulated as unicast IP packets (Protocol 89) between the transit interface IPs (192.168.3.1 and 192.168.3.2).

DoNotAge (DNA) Flag: LSAs flooded across the Virtual Link have the DNA bit set to suppress periodic 30-minute LSA refreshes.

Dual-ABR Promotion: Establishing VL0 promotes R7 into a full Area 0 ABR. Consequently, any inter-area route summarization for Area 2 (area 2 range) must be applied identically on both R3 and R7.

4. Verification & Troubleshooting
   Cisco CLI
   R7# show ip ospf interface brief
   Interface PID Area IP Address/Mask Cost State Nbrs F/C
   VL0 1 0 192.168.3.2/24 10 P2P 1/1
   Et0/2 1 0 172.16.20.1/24 10 BDR 1/1
   Et0/0 1 2 192.168.3.2/24 10 P2P 1/1
   Failure Scenario: Virtual Link stuck in DOWN.

Root Cause: Virtual Links bind strictly to the OSPF Router ID, not the physical interface IP, and cannot traverse a Stub or NSSA area. Verify that 3.3.3.3 and 7.7.7.7 have intra-area reachability inside Area 2 (show ip ospf border-routers).
