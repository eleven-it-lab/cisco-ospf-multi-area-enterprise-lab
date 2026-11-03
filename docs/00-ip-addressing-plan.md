# 00 - IP Addressing & Subnet Allocation Plan

## 1. Global Addressing Strategy

This enterprise core architecture utilizes a dual-stack approach, migrating legacy IPv4 segments to a modern IPv6 architecture while maintaining symmetrical traffic engineering.

- **IPv4 Core Infrastructure:** `172.16.0.0/16` and `192.168.0.0/16` (RFC 1918 Private Space)
- **IPv6 Global Unicast Address (GUA) Block:** `2001:DB8:ACAD::/48` (RFC 3849 Documentation Prefix)
- **IPv6 Link-Local Standard:** `FE80::X` (Where `X` is the Router ID / Node Number)

---

## 2. Router ID & Loopback Management

Loopback interfaces provide stable OSPF Router IDs (RIDs) and BGP peering endpoints. In OSPFv3, the IPv4 Router ID is strictly retained to build the Shortest Path First (SPF) tree.

| Node    | OSPF Router ID | IPv4 Loopback0 (`/32`) | IPv6 Loopback0 (`/128`)  |
| :------ | :------------- | :--------------------- | :----------------------- |
| **R1**  | `1.1.1.1`      | `1.1.1.1`              | `2001:DB8:ACAD:1::1`     |
| **R2**  | `2.2.2.2`      | `2.2.2.2`              | `2001:DB8:ACAD:2::2`     |
| **R3**  | `3.3.3.3`      | `3.3.3.3`              | `2001:DB8:ACAD:3::3`     |
| **R4**  | `4.4.4.4`      | `4.4.4.4`              | `2001:DB8:ACAD:4::4`     |
| **R5**  | `5.5.5.5`      | `5.5.5.5`              | `2001:DB8:ACAD:5::5`     |
| **R6**  | `6.6.6.6`      | `6.6.6.6`              | _(IPv4 Only Branch)_     |
| **R7**  | `7.7.7.7`      | `7.7.7.7`              | _(IPv4 Only Transit)_    |
| **R8**  | `8.8.8.8`      | `8.8.8.8`              | _(IPv4 Only Core Edge)_  |
| **R9**  | `9.9.9.9`      | `9.9.9.9`              | _(IPv4 Only NBMA Spoke)_ |
| **R10** | `10.10.10.10`  | `10.10.10.10`          | _(IPv4 Only NBMA Spoke)_ |

---

## 3. Core Infrastructure Links (By OSPF Area)

### Area 0: Backbone Core (Dual-Stack)

The primary Area 0 broadcast segment connects R1, R2, R3, and R4.

- **IPv4 Subnet:** `172.16.10.0/24`
- **IPv6 Subnet:** `2001:DB8:ACAD:10::/64`
- **Node Mapping:** `.1`/`::1` (R1), `.2`/`::2` (R2), `.3`/`::3` (R3), `.4`/`::4` (R4)

### Area 0: Discontiguous Edge

Connects Transit ABR R7 to the isolated Area 0 edge router R8.

- **IPv4 Subnet:** `172.16.20.0/24` (R7 = `.1`, R8 = `.2`)

### Area 51: Standard NSSA (Dual-Stack)

Point-to-point links connecting the NSSA ASBR (R5) to the Core ABRs (R2 & R4).

- **R2 to R5 Link:**
  - IPv4: `192.168.2.0/24` (R2 = `.1`, R5 = `.2`)
  - IPv6: `2001:DB8:ACAD:25::/64` (R2 = `::2`, R5 = `::5`)
- **R4 to R5 Link:**
  - IPv4: `192.168.4.0/24` (R4 = `.1`, R5 = `.2`)
  - IPv6: `2001:DB8:ACAD:45::/64` (R4 = `::4`, R5 = `::5`)

### Area 2: Transit Area

Carries the OSPF Virtual Link connecting the partitioned backbone.

- **Transit Link (R3 to R7):** `192.168.3.0/24` (R3 = `.1`, R7 = `.2`)
- **Host Subnet (R7 to H1):** `192.168.7.0/24` (R7 = `.1`, H1 = `.100`)

### Area 3: Legacy NBMA WAN

Point-to-Multipoint Non-Broadcast segment anchored at Hub R3.

- **NBMA Subnet:** `192.168.9.0/24`
- **Node Mapping:** R3 = `.3`, R9 = `.9`, R10 = `.10`

---

## 4. Branch & External Subnets (Summarization Targets)

### Area 2: Transit Branch Subnets

Simulated branch networks originating on R7. These are summarized at the ABRs (R3 and R7) into a single `/22` aggregate (`172.16.16.0/22`) before entering Area 0.

- **Loopback 1:** `172.16.16.1/24`
- **Loopback 2:** `172.16.17.1/24`
- **Loopback 3:** `172.16.18.1/24`

### Non-OSPF External Subnets

Simulated external customer networks originating on R6, redistributed into Area 51 by ASBR R5. These are summarized at R5 into a single `/18` aggregate (`172.16.0.0/18`).

- **Transit (R5 to R6):** `192.168.56.0/24` (R5 = `.5`, R6 = `.6`)
- **Customer Subnet 1:** `172.16.30.0/24`
- **Customer Subnet 2:** `172.16.31.0/24`
- **Customer Subnet 3:** `172.16.32.0/24`
- **Customer Subnet 4:** `172.16.33.0/24`
