````markdown
# 05 - Asymmetric Traffic Engineering & Load Balancing

## 1. Engineering Requirement

In Area 51, `R5` is dual-homed to ABRs `R2` (`Ethernet0/0`) and `R4` (`Ethernet0/1`). Rather than letting OSPF either load-balance everything or leave one link completely idle, we engineer a split policy:

1. **Outbound Default Route (`0.0.0.0/0` & `::/0`):** Strictly exit via `R2` (`Ethernet0/0`), using `R4` (`Ethernet0/1`) only as a hot-standby failover path.
2. **Internal Inter-Area Traffic (`172.16.10.0/24` & `2001:DB8:ACAD:10::/64`):** Utilize Equal-Cost Multi-Path (ECMP) load-balancing across both `R2` and `R4` simultaneously.

---

## 2. Decoupling Default Route Preference from Interface Cost

If we increased the interface cost on `R5`'s `Ethernet0/1` link (`ip ospf cost 500`), we would shift default traffic to `R2`, **but we would also break ECMP load-balancing for all internal `O IA` routes.**

Instead, we leave the physical interface costs symmetrical (`Cost 10` on both links) and manipulate the **Type 7 NSSA Default Route Seed Metric** at the ABRs:

- **On `R2`:** Originate default route as Type 2 (`N2`/`ON2`) with a metric of `10`.
- **On `R4`:** Originate default route as Type 2 (`N2`/`ON2`) with a metric of `50`.

Because **External Type 2 (`N2`)** metrics ignore internal OSPF interface costs unless there is a tie, `R5` installs `R2` as the sole gateway of last resort (`Metric 10 < Metric 50`), while `O IA` / `OI` routes calculate the symmetrical physical interface costs (`10 + 10 = 20`) and install both paths into the RIB/FIB.

---

## 3. Configuration (Dual-Stack IPv4 & IPv6)

```cisco
! On R2 (Primary Default Exit - Metric 10)
router ospf 1
 area 51 nssa default-information-originate metric 10
ipv6 router ospf 1
 area 51 nssa default-information-originate metric 10

! On R4 (Secondary Default Exit - Metric 50)
router ospf 1
 area 51 nssa default-information-originate metric 50
ipv6 router ospf 1
 area 51 nssa default-information-originate metric 50
```

## Verification

```
R5# show ipv6 route ospf
ON2 ::/0 [110/10]
via 2001:DB8:ACAD:25::2, Ethernet0/0
OI 2001:DB8:ACAD:10::/64 [110/20]
via FE80::2, Ethernet0/0
via FE80::4, Ethernet0/1
```
````
