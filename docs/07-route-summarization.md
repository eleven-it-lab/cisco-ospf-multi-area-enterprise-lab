````markdown
# 07 - OSPF Route Summarization & The Multi-ABR Leak Trap

## 1. Where Summarization Occurs in OSPF

Because Type 1 and Type 2 LSAs cannot be modified inside an area, OSPF summarization can only be configured at LSA translation boundaries:

1. **ASBR External Summarization (`summary-address`):** Aggregates redistributed external routes (Type 5 or Type 7 LSAs) at the point of injection.
2. **ABR Inter-Area Summarization (`area <id> range`):** Aggregates internal Type 1/2 prefixes into a single Type 3 Summary LSA before flooding into adjacent areas.

---

## 2. ASBR Summarization on `R5` (Area 51)

`R5` redistributes four contiguous external subnets (`172.16.30.0/24` through `172.16.33.0/24`). We collapse these into a single `/18` aggregate (`172.16.0.0/18`):

```cisco
! On R5
router ospf 1
 summary-address 172.16.0.0 255.255.192.0
```

## ABR Summarization & Solving Two Classic CCIE Traps (Area 2)

# Trap 1: Loopback /32 Host-Route Stub Advertisement

```
! On R7
interface range Loopback1 - 3
 ip ospf network point-to-point
```

# Trap 2: The Multi-ABR Route Leak (Virtual Link Side-Effect)

```
! Must be applied on BOTH R3 and R7
router ospf 1
 area 2 range 172.16.16.0 255.255.252.0
```
````
