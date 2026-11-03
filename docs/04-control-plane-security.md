````markdown
# 04 - Control Plane Hardening: GTSM (TTL Security) & HMAC-SHA-256

## 1. Cryptographic Authentication (HMAC-SHA-256)

Legacy OSPF authentication modes (`ip ospf authentication-key` plaintext and `ip ospf message-digest-key` MD5) are vulnerable to packet sniffing and collision attacks.

In Area 51 (`R2`, `R4`, `R5`), control-plane packets are authenticated using **IOS Key Chains** with **HMAC-SHA-256** (`cryptographic-algorithm hmac-sha-256`), defined in RFC 5709.

```cisco
! Configure identically on R2, R4, and R5
key chain OSPF_KEY
 key 1
  key-string ELEVEN_CORE_SEC
  cryptographic-algorithm hmac-sha-256

! Apply to Area 51 interfaces (R2 Et0/1, R4 Et0/1, R5 Et0/0 & Et0/1)
interface Ethernet0/0
 ip ospf authentication key-chain OSPF_KEY
```
````
