````markdown
# 03 - Legacy NBMA WAN: Point-to-Multipoint Non-Broadcast

## 1. Architectural Challenge in Area 3

Area 3 connects Hub router `R3` (`192.168.9.3`) to Spoke routers `R9` (`192.168.9.9`) and `R10` (`192.168.9.10`) over a shared `/24` non-broadcast multi-access (NBMA) segment (`192.168.9.0/24`) where multicast (`224.0.0.5`) is unsupported and direct spoke-to-spoke Layer 2 adjacency may be restricted.

If left in default `BROADCAST` or `NON_BROADCAST` mode:

- OSPF performs a DR/BDR election. If a spoke (`R9` or `R10`) accidentally wins the DR election, the topology breaks.
- OSPF assumes full Layer 2 mesh connectivity and leaves the original spoke next-hop IP unchanged, causing spoke-to-spoke packets to fail.

---

## 2. Point-to-Multipoint Non-Broadcast Solution

Configuring `ip ospf network point-to-multipoint non-broadcast` transforms the shared `/24` segment into a collection of logical point-to-point links anchored at `R3`:

1. **No DR/BDR Election:** Adjacencies transition directly to `FULL/-`.
2. **Unicast Hellos:** Because `non-broadcast` is specified, neighbors are statically defined on the hub (`R3`) using `neighbor <ip>`.
3. **Automatic `/32` Host Route Generation:** Every router injects a `/32` host route for its own NBMA interface (`192.168.9.3/32`, `192.168.9.9/32`, `192.168.9.10/32`). When `R9` sends traffic to `R10`, `R3` rewrites the next-hop to itself (`192.168.9.3`), seamlessly hair-pinning spoke-to-spoke traffic through the hub.

---

## 3. Configuration

```cisco
! On Hub R3
interface Ethernet0/2
 ip address 192.168.9.3 255.255.255.0
 ip ospf network point-to-multipoint non-broadcast
 ip ospf 1 area 3
!
router ospf 1
 neighbor 192.168.9.9
 neighbor 192.168.9.10

! On Spokes R9 & R10
interface Ethernet0/0
 ip address 192.168.9.10 255.255.255.0
 ip ospf network point-to-multipoint non-broadcast
 ip ospf 1 area 3
```
````
