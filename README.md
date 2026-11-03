# Enterprise OSPFv2 & OSPFv3 Dual-Stack Core Architecture

![Cisco IOS](https://img.shields.io/badge/Cisco-IOS_15.x-blue.svg)
![EVE-NG](https://img.shields.io/badge/EVE--NG-Emulation-orange.svg)
![IPv6](https://img.shields.io/badge/IPv6-Dual_Stack-success.svg)

## Project Overview

This repository documents a CCNP/CCIE-level enterprise routing architecture utilizing OSPFv2 (IPv4) and OSPFv3 (IPv6). The lab environment replicates a complex ISP and enterprise core, demonstrating advanced routing behaviors including asymmetrical traffic engineering, control plane hardening, legacy NBMA integration, and dual-stack Link-Local peering.

_Attribution: This topology is heavily adapted and expanded from [NetworkLessons OSPF Professional Lab 1](https://networklessons.com/labs/ospf-professional-lab-1), with original engineering configurations, IPv6 integration, and traffic engineering policies designed by ELEVEN IT LAB._

## Technical Objectives Achieved

1. **Multi-Area Architecture & Scaling:** Configured Backbone (Area 0), Standard NSSA (Area 51), Transit (Area 2), and NBMA (Area 3) zones to strictly contain Link-State Advertisements (LSAs).
2. **Control Plane Security:** Deployed Generalized TTL Security Mechanism (GTSM) and HMAC-SHA-256 cryptographic authentication to protect Area 51 peering.
3. **Asymmetric Traffic Engineering:** Manipulated directional interface costs and Type 7 NSSA default route metrics to separate outbound Internet traffic (via R2) from internal load-balanced inter-area traffic (via R2 & R4).
4. **Discontiguous Backbone Recovery:** Engineered an OSPF Virtual Link across Area 2 to stitch a partitioned Area 0 segment back into the core.
5. **Advanced Route Filtering:** Demonstrated the architectural difference between LSDB-level filtering (ABR `filter-list`) and RIB-level filtering (local `distribute-list`).
6. **Dual-Stack IPv6 Migration:** Deployed OSPFv3 natively over `FE80::` Link-Local addresses, decoupling the SPF topology tree from the Global Unicast Addressing scheme.

## Topology Architecture

_(Insert Mermaid Diagram or reference to `/topology/ospf-topology.png` here)_

### Router Roles & Inventory

| Hostname     | Role        | Connected Areas        | Key Technologies Implemented                        |
| :----------- | :---------- | :--------------------- | :-------------------------------------------------- |
| **R1**       | Core Router | Area 0                 | DR/BDR Election, OSPFv3 Dual-Stack                  |
| **R2**       | ABR         | Area 0, Area 51        | NSSA Type 7 Injection, GTSM, SHA-256, IPv6          |
| **R3**       | ABR / Hub   | Area 0, Area 2, Area 3 | Virtual Link, NBMA P2MP, ABR Filtering, IPv6        |
| **R4**       | ABR         | Area 0, Area 51        | Asymmetric Routing Cost Manipulation, IPv6          |
| **R5**       | ASBR        | Area 51, External      | External Route Redistribution, ASBR Summarization   |
| **R7**       | Transit ABR | Area 2, Area 0         | Discontiguous Backbone bridging via Virtual Link    |
| **R9 / R10** | Spokes      | Area 3                 | Point-to-Multipoint Non-Broadcast, Distribute-Lists |

## Repository Structure & Documentation Guide

The project is divided into modular technical documentation explaining the _why_ behind the configurations.

- `/configs/`: Raw Cisco IOS startup configurations exported directly from EVE-NG.
- `/docs/`: Deep-dive technical explanations of the implemented protocols.
  - `04-control-plane-security.md`: Mechanics of TTL-Security matching and Key Chains.
  - `05-traffic-engineering.md`: Directional cost modification and Type 7 LSA manipulation.
  - `08-ospfv3-dual-stack.md`: Why OSPFv3 utilizes Type 8/9 LSAs and Link-Local peering.
- `/verification/`: Console captures proving the architecture works (e.g., Database analysis, asymmetric routing tables).

## Operational Verification

Crucial commands utilized in this deployment to verify LSDB synchronization and RIB installation:

```bash
# Verify LSDB filtering vs RIB filtering
show ip ospf database summary 5.5.5.5
show ip route 5.5.5.5

# Verify NSSA Type 7 default routes and load balancing
show ip route ospf

# Verify OSPFv3 Link-Local Decoupling
show ipv6 ospf neighbor
```
