````markdown
# Production RIB Captures & Troubleshooting Playbook

## 1. Core Router `R4` OSPFv2 Routing Table (Post-Summarization)

Verifies:

1. Default route `0.0.0.0/0` learned via `R1` (`172.16.10.1`).
2. Partitioned Area 0 router `8.8.8.8/32` learned as an intra-area `O` route via `R3` (`VL0`).
3. ASBR external summary `172.16.0.0/18` (`O N2`) and ABR inter-area summary `172.16.16.0/22` (`O IA`) cleanly suppressing component `/24` and `/32` prefixes.
4. Area 3 NBMA `/32` host routes (`192.168.9.3`, `.9`, `.10`) injected by `point-to-multipoint non-broadcast`.

```text
R4#show ip route ospf
Gateway of last resort is 172.16.10.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.10.1, 03:04:15, Ethernet0/0
      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/11] via 172.16.10.1, 03:04:15, Ethernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/11] via 172.16.10.2, 03:04:12, Ethernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/11] via 172.16.10.3, 03:04:12, Ethernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/11] via 192.168.4.2, 03:04:51, Ethernet0/1
      6.0.0.0/32 is subnetted, 1 subnets
O N2     6.6.6.6 [110/20] via 192.168.4.2, 03:04:51, Ethernet0/1
      7.0.0.0/32 is subnetted, 1 subnets
O IA     7.7.7.7 [110/21] via 172.16.10.3, 03:04:12, Ethernet0/0
      8.0.0.0/32 is subnetted, 1 subnets
O        8.8.8.8 [110/31] via 172.16.10.3, 03:04:12, Ethernet0/0
      9.0.0.0/32 is subnetted, 1 subnets
O IA     9.9.9.9 [110/21] via 172.16.10.3, 03:03:59, Ethernet0/0
      10.0.0.0/32 is subnetted, 1 subnets
O IA     10.10.10.10 [110/21] via 172.16.10.3, 03:03:59, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 3 subnets, 3 masks
O N2     172.16.0.0/18 [110/20] via 192.168.4.2, 00:17:43, Ethernet0/1
O IA     172.16.16.0/22 [110/21] via 172.16.10.3, 00:08:06, Ethernet0/0
O        172.16.20.0/24 [110/30] via 172.16.10.3, 00:08:06, Ethernet0/0
O     192.168.2.0/24 [110/20] via 192.168.4.2, 00:08:06, Ethernet0/1
O IA  192.168.3.0/24 [110/20] via 172.16.10.3, 00:08:06, Ethernet0/0
O IA  192.168.7.0/24 [110/30] via 172.16.10.3, 00:08:06, Ethernet0/0
      192.168.9.0/32 is subnetted, 3 subnets
O IA     192.168.9.3 [110/10] via 172.16.10.3, 00:08:06, Ethernet0/0
O IA     192.168.9.9 [110/20] via 172.16.10.3, 00:08:06, Ethernet0/0
O IA     192.168.9.10 [110/20] via 172.16.10.3, 00:08:06, Ethernet0/0
```
````
