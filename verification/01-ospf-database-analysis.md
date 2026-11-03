````markdown
# OSPF Link-State Database (LSDB) & Adjacency Verification

## 1. Area 2 Transit & Virtual-Link State (`R7`)

Confirms `VL0` is `P2P` in Area 0, `Et0/0` is `P2P` in Area 2, and `Lo1–Lo3` are operating in `P2P` mode to advertise true `/24` masks for ABR summarization:

```text
R7#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
VL0          1     0               192.168.3.2/24     10    P2P   1/1
Et0/2        1     0               172.16.20.1/24     10    BDR   1/1
Lo0          1     2               7.7.7.7/32         1     LOOP  0/0
Lo3          1     2               172.16.18.1/24     1     P2P   0/0
Lo2          1     2               172.16.17.1/24     1     P2P   0/0
Lo1          1     2               172.16.16.1/24     1     P2P   0/0
Et0/1        1     2               192.168.7.1/24     10    DR    0/0
Et0/0        1     2               192.168.3.2/24     10    P2P   1/1
```
````
