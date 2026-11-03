````markdown
# 02 - Standard NSSA Design & Type 7-to-Type 5 LSA Translation

## 1. Why Standard NSSA for Area 51?

Area 51 connects core ABRs `R2` and `R4` to `R5`. However, `R5` also acts as an Autonomous System Boundary Router (ASBR), redistributing external prefixes (`6.6.6.6/32` and `172.16.30.0/24–172.16.33.0/24`) from non-OSPF branch router `R6`.

- A **Standard Stub** area rejects Type 5 External LSAs from the core, but forbids `R5` from redistributing `R6`'s external routes.
- A **Not-So-Stubby Area (NSSA)** blocks Type 5 External LSAs from entering Area 51 while allowing `R5` to inject external prefixes locally as **Type 7 NSSA External LSAs** (`O N2`).

---

## 2. Configuration Blueprint

Unlike a Stub area, a Standard NSSA **does not** automatically originate a default route at the ABRs. You must explicitly append `default-information-originate` on the ABRs (`R2` and `R4`) to inject a Type 7 default route (`0.0.0.0/0`) into Area 51.

```cisco
! On R2 (Primary Exit ABR)
router ospf 1
 area 51 nssa default-information-originate metric 10

! On R4 (Secondary Backup ABR)
router ospf 1
 area 51 nssa default-information-originate metric 50

! On R5 (NSSA ASBR)
router ospf 1
 area 51 nssa
 redistribute static subnets
```
````
