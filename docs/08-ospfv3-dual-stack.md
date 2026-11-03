``markdown

# 08 - OSPFv3 Dual-Stack Migration & Link-Local Architecture

## 1. Architectural Shift from OSPFv2 to OSPFv3 (RFC 5340)

OSPFv3 is not merely OSPFv2 with 128-bit addresses; it restructures how the SPF tree is separated from prefix reachability:

1. **Link-Local Peering (`FE80::`):** All OSPFv3 adjacencies and next-hop forwarding attributes use IPv6 Link-Local addresses (`FE80::/10`). Global Unicast Addresses (GUAs) can be renumbered without dropping OSPF neighbors.
2. **32-Bit Router ID Retention:** OSPFv3 still requires a 32-bit dotted-decimal Router ID (`1.1.1.1` through `5.5.5.5`) to identify nodes in the SPF graph.
3. **Separation of Topology and Prefixes (Type 8 & Type 9 LSAs):**
   - In OSPFv3, **Type 1 (Router)** and **Type 2 (Network)** LSAs contain **zero IPv6 addresses**—they only describe the pure topological graph.
   - **Type 8 (Link LSA):** Has link-local flooding scope; advertises the router's `FE80::` address and local prefixes to neighbors on the wire.
   - **Type 9 (Intra-Area Prefix LSA):** Attaches global IPv6 subnets (`2001:DB8:ACAD::/64`) to the Type 1/2 SPF tree so prefix renumbering does not trigger a full SPF recalculation.

---

## 2. Dual-Stack IPv6 Addressing Plan (Area 0 & Area 51)

| Router | Loopback0 (`/128`)   | Area 0 `Et0/0` (`2001:DB8:ACAD:10::/64`) | Area 51 Links (`/64`)                 | Link-Local |
| :----- | :------------------- | :--------------------------------------- | :------------------------------------ | :--------- |
| **R1** | `2001:DB8:ACAD:1::1` | `2001:DB8:ACAD:10::1`                    | —                                     | `FE80::1`  |
| **R2** | `2001:DB8:ACAD:2::2` | `2001:DB8:ACAD:10::2`                    | `Et0/1`: `2001:DB8:ACAD:25::2`        | `FE80::2`  |
| **R3** | `2001:DB8:ACAD:3::3` | `2001:DB8:ACAD:3::3`                     | —                                     | `FE80::3`  |
| **R4** | `2001:DB8:ACAD:4::4` | `2001:DB8:ACAD:10::4`                    | `Et0/1`: `2001:DB8:ACAD:45::4`        | `FE80::4`  |
| **R5** | `2001:DB8:ACAD:5::5` | —                                        | `Et0/0`: `:25::5` / `Et0/1`: `:45::5` | `FE80::5`  |

---

## 3. Configuration (`R5` NSSA Spoke Example)

```cisco
ipv6 unicast-routing
!
interface Loopback0
 ipv6 address 2001:DB8:ACAD:5::5/128
 ipv6 ospf 1 area 51
!
interface Ethernet0/0
 ipv6 address 2001:DB8:ACAD:25::5/64
 ipv6 address FE80::5 link-local
 ipv6 ospf 1 area 51
!
interface Ethernet0/1
 ipv6 address 2001:DB8:ACAD:45::5/64
 ipv6 address FE80::5 link-local
 ipv6 ospf 1 area 51
!
ipv6 router ospf 1
 router-id 5.5.5.5
 area 51 nssa
```
