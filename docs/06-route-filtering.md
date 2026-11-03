````markdown
# 06 - OSPF Route Filtering: ABR `filter-list` (LSDB) vs. Local `distribute-list` (RIB)

## 1. The Unbreakable Law of OSPF Filtering

Every router within the same OSPF area must maintain an **identical Link-State Database (LSDB)**. If a single internal router blocks an LSA from entering its database while its neighbors retain it, the Dijkstra SPF trees diverge and routing loops occur.

Consequently, OSPF supports two fundamentally different filtering mechanisms:

1. **ABR Type 3 LSA Filtering (`area filter-list`):** Blocks the ABR from generating a Type 3 Summary LSA at the area border. The prefix is completely absent from the downstream area's LSDB and RIB.
2. **Local RIB Filtering (`distribute-list in`):** Permits the LSA into the local LSDB and floods it normally to downstream neighbors, but intercepts the prefix before the OSPF process installs it into the local router's Routing Table (RIB).

---

## 2. Implementation Scenarios

### Scenario A: Block `4.4.4.4/32` from Entering Area 3 Altogether (`R3` ABR)

Configured on `R3` (the Area 3 ABR) using `area 3 filter-list prefix ... in`:

```cisco
ip prefix-list BLOCK_R4 seq 5 deny 4.4.4.4/32
ip prefix-list BLOCK_R4 seq 10 permit 0.0.0.0/0 le 32
!
router ospf 1
 area 3 filter-list prefix BLOCK_R4 in
```
````
