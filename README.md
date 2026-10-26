<div align="center">

# 🔐 EIGRP MD5 Authentication

### Stop Rogue Routers · Secure Neighbor Adjacency · Key Chains — Cisco EIGRP AS 1

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-MD5%20Authentication-critical?style=for-the-badge)
![AS](https://img.shields.io/badge/AS-1-orange?style=for-the-badge)
![Key](https://img.shields.io/badge/Key%20Chain-khan-yellow?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

**❌ Neighbor down → authentication mode changed  ➜  ✅ Neighbor up → new adjacency**

</div>

---

## 📖 Table of Contents

[Overview](#-project-overview) · [Lab Snapshot](#-lab-snapshot) · [Topology](#️-network-topology) · [IP Table](#-ip-addressing-table) · [Configuration](#️-configuration) · [Stage-by-Stage Verification](#-verification--stage-by-stage) · [How MD5 Works](#-how-eigrp-md5-authentication-works) · [Command Breakdown](#-command-breakdown) · [Troubleshooting](#-troubleshooting-guide) · [Advanced](#-advanced-topics) · [Takeaways](#-key-takeaways) · [Challenges](#-challenges)

---

## 📌 Project Overview

By default, **EIGRP trusts any router** that sends packets with the right AS number on a shared link. A rogue or misconfigured router can become a neighbor in seconds and **inject fake routes**, poisoning the routing table.

This lab fixes that. Two Cisco 2911 routers (**R1** and **R2**) are connected over a serial link (**1.1.1.0/24**), each with its own LAN. I secured the adjacency with **MD5 authentication using a key chain** and watched the neighbor relationship **break and recover** depending on whether the keys matched.

> 🎯 **The big idea:** if the key doesn't match, the adjacency doesn't exist. No key, no routes.

---

## 📸 Lab Snapshot

| Item | Value |
|:-----|:------|
| 🧰 Tool | Cisco Packet Tracer |
| 🖧 Routers | 2 × Cisco 2911 (R1, R2) |
| 🔀 Switches / PCs | 2 × 2950-24 / 6 × PC-PT |
| 🌐 Routing protocol | EIGRP **AS 1** |
| 🔗 Secured link | R1 Se0/0/0 (1.1.1.1) ↔ R2 Se0/0/0 (1.1.1.2) |
| 🔐 Authentication type | **MD5** |
| 🗝️ Key chain name | `khan` |
| 🔢 Key ID | **1** |
| 🔑 Key string | `cisco` *(lab value)* |
| 📣 Key events | `authentication mode changed` → `new adjacency` |

---

## 🎬 What Happens in This Lab (Timeline)

```
 ① BEFORE            ② R1 SECURED ONLY               ③ BOTH SECURED
 ───────────         ───────────────────             ───────────────────
 R1 ◄──────► R2      R1 🔐 ──✖── R2                  R1 🔐 ◄════► 🔐 R2
 plain EIGRP         MD5 on R1, none on R2           same key on both
 adjacency UP        adjacency DOWN ❌                adjacency UP ✅
                     "authentication mode changed"   "new adjacency"
```

| Stage | R1 | R2 | Adjacency | Log message |
|:-----:|:--:|:--:|:---------:|:------------|
| ① | No auth | No auth | ✅ Up | *(normal EIGRP neighbor)* |
| ② | **MD5 enabled** | No auth | ❌ **Down** | `Neighbor 1.1.1.2 (Serial0/0/0) is down: authentication mode changed` |
| ③ | MD5 + key chain | **MD5 + same key chain** | ✅ **Up** | `Neighbor 1.1.1.1 (Serial0/0/0) is up: new adjacency` |

---

## 🎯 Objectives

- ✅ Create a **key chain** with key ID and key string on both routers
- ✅ Enable **MD5 authentication** for EIGRP AS 1 on the serial interface
- ✅ **Bind** the key chain to the interface
- ✅ Watch the adjacency **drop** when only one side is secured
- ✅ Watch the adjacency **recover** when both sides match
- ✅ Confirm the result in the **running-config**
- ✅ Understand what MD5 protects (and what it does **not**)

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                                192.168.2.0/24

PC1 ─┐                          1.1.1.0/24                              ┌─ PC4
PC2 ─┼─ Switch0 ── Gi0/0 ┌──────┐ Se0/0/0 ═══════ Se0/0/0 ┌──────┐ Gi0/0 ── Switch1 ─┼─ PC5
PC3 ─┘  (2950)    .1     │  R1  │ .1      🔐 MD5      .2   │  R2  │  .1    (2950)      └─ PC6
                        │ 2911 │                          │ 2911 │
                        └──────┘                          └──────┘
                    key chain "khan"                  key chain "khan"
                    key 1 · cisco                     key 1 · cisco
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Mask | Connected To |
|:------:|:---------:|:----------:|:----:|:------------:|
| **R1** | Gi0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (LAN 1) |
| **R1** | Se0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 Se0/0/0 🔐 |
| **R2** | Gi0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (LAN 2) |
| **R2** | Se0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 Se0/0/0 🔐 |
| PC1 / PC2 / PC3 | NIC | 192.168.1.2 / .3 / .4 | 255.255.255.0 | Switch0 |
| PC4 / PC5 / PC6 | NIC | 192.168.2.2 / .3 / .4 | 255.255.255.0 | Switch1 |

---

## 🔑 Authentication Parameters (What Must Match)

| Parameter | Value | Must match on both routers? |
|:----------|:-----:|:---------------------------:|
| EIGRP AS number | **1** | ✅ Yes |
| Authentication mode | **md5** | ✅ Yes |
| Key ID | **1** | ✅ **Yes** |
| Key string | `cisco` | ✅ **Yes** |
| Key chain name | `khan` | ❌ No — local label only |
| Applied on | Serial0/0/0 | On **each** side of the link |

---

## ⚙️ Configuration

> 💡 EIGRP AS 1 was already running between the routers. This lab **adds security on top of it**.

### 🔹 Step 1 — Create the key chain *(R1 and R2)*

```bash
key chain khan
 key 1
  key-string cisco
```

### 🔹 Step 2 — Enable MD5 on the interface *(R1 and R2)*

```bash
interface Serial0/0/0
 ip authentication mode eigrp 1 md5
```

### 🔹 Step 3 — Bind the key chain to the interface *(R1 and R2)*

```bash
interface Serial0/0/0
 ip authentication key-chain eigrp 1 khan
```

### 🔹 Step 4 — Save

```bash
do write memory
```

<details>
<summary><b>📜 Click to see the full command flow exactly as typed in the lab</b></summary>

**R1**
```
R1(config)# key chain khan
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string cisco
R1(config-keychain-key)# exit
R1(config-keychain)# do wr
Building configuration...
[OK]
R1(config-keychain)# exit
R1(config)# interface serial 0/0/0
R1(config-if)# ip authentication mode eigrp 1 md5
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 1.1.1.2 (Serial0/0/0) is down: authentication mode changed
R1(config-if)# do wr
R1(config-if)# ip authentication key-chain eigrp 1 khan
R1(config-if)# do wr
```

**R2**
```
R2(config)# key chain khan
R2(config-keychain)# key 1
R2(config-keychain-key)# key-string cisco
R2(config-keychain-key)# exit
R2(config)# interface serial 0/0/0
R2(config-if)# ip authentication mode eigrp 1 md5
R2(config-if)# ip authentication key-chain eigrp 1 khan
%DUAL-5-NBRCHANGE: IP-EIGRP 1: Neighbor 1.1.1.1 (Serial0/0/0) is up: new adjacency
R2(config-if)# do wr
```

> 📝 A mistyped command on R2 (`% Invalid input detected at '^' marker`) was harmless — it was corrected and the configuration continued.

</details>

---

## ✅ Verification — Stage by Stage

### 🟡 Stage 1: R1 Enables MD5 → Adjacency Drops ❌

```
R1(config)# interface serial 0/0/0
R1(config-if)# ip authentication mode eigrp 1 md5
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 1.1.1.2 (Serial0/0/0) is down: authentication mode changed   ◄── 🔥
```

🧠 **Why?** R1 now signs every EIGRP packet with an MD5 digest. R2 still sends plain packets, so R1 can no longer trust them — the neighbor is torn down.

---

### 🟢 Stage 2: R2 Configured with the Same Key → Adjacency Returns ✅

```
R2(config)# interface serial 0/0/0
R2(config-if)# ip authentication mode eigrp 1 md5
R2(config-if)# ip authentication key-chain eigrp 1 khan
%DUAL-5-NBRCHANGE: IP-EIGRP 1: Neighbor 1.1.1.1 (Serial0/0/0) is up: new adjacency                      ◄── 🔥
```

🧠 **Why?** Both routers now use the same key ID (1) and key string (`cisco`). Digests validate, packets are accepted, and the adjacency is rebuilt.

---

### 🔵 Stage 3: Running-Config Proof (R1) 📄

```
key chain khan                                          ◄── 🗝️ key chain created
 key 1
  key-string cisco
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
!
interface Serial0/0/0
 ip address 1.1.1.1 255.255.255.0
 ip authentication mode eigrp 1 md5                     ◄── 🔐 MD5 enabled for AS 1
 ip authentication key-chain eigrp 1 khan               ◄── 🔗 bound to key chain "khan"
 clock rate 64000
!
interface Serial0/0/1
 no ip address
 shutdown
```

---

### 🧾 Evidence Summary

| # | Evidence | What it proves |
|:-:|:---------|:---------------|
| 1 | R1 key chain commands | Key chain `khan`, key 1, key-string created |
| 2 | R1 `authentication mode changed` | MD5 on one side **breaks** the old adjacency |
| 3 | R2 `new adjacency` | Matching key **restores** the adjacency |
| 4 | R1 running-config | Authentication lines are on Serial0/0/0 |

### 📷 Add These Final Screenshots to Complete the Lab

```bash
show ip eigrp neighbors       # 1.1.1.x via Se0/0/0 is back
show ip route eigrp           # D 192.168.x.0/24 learned again
show key chain                # key 1 present
ping 192.168.2.2              # LAN 1 → LAN 2 works after securing EIGRP
```

---

## 🔍 How EIGRP MD5 Authentication Works

```
 R1 (sender)                                               R2 (receiver)
 ┌─────────────────────────────┐                ┌─────────────────────────────┐
 │ EIGRP packet                │                │ 1. Read key ID from packet  │
 │   +  key ID 1               │   packet +     │ 2. Find key 1 in its chain  │
 │   +  key-string "cisco"     │ ─────────────► │ 3. Recompute MD5 digest     │
 │   ▼                         │   MD5 digest   │ 4. Compare digests          │
 │ MD5 digest calculated       │                │    ✅ same  → accept        │
 └─────────────────────────────┘                │    ❌ differ → DROP         │
                                                └─────────────────────────────┘
```

- 🔸 The **key string is never sent** across the wire — only the digest
- 🔸 **All** EIGRP packet types are protected: Hello, Update, Query, Reply, Ack
- 🔸 A wrong key → packets are **silently discarded** → **no adjacency**

### 🛡️ Security Impact

| Threat | Without MD5 | With MD5 |
|:-------|:-----------:|:--------:|
| Rogue router becomes a neighbor | ❌ Possible | ✅ Blocked |
| Fake route injection | ❌ Possible | ✅ Blocked |
| Packet tampering in transit | ❌ Undetected | ✅ Detected |
| Misconfigured device joins by accident | ❌ Possible | ✅ Blocked |
| Eavesdropping on routes | ❌ Visible | ❌ **Still visible** (MD5 authenticates, it does not encrypt) |

---

## 🧩 Command Breakdown

| Command | What it does |
|:--------|:-------------|
| `key chain khan` | Creates a key chain named **khan** (local name) |
| `key 1` | Defines key **ID 1** — the ID is sent in the packet |
| `key-string cisco` | Sets the shared **secret** for key 1 |
| `ip authentication mode eigrp 1 md5` | Turns on **MD5** for **AS 1** on this interface |
| `ip authentication key-chain eigrp 1 khan` | Tells the interface **which key chain** to use for AS 1 |
| `do write memory` | Saves the configuration from any config mode |

---

## 🧰 Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|:--------|:-------------|:----|
| Neighbor down: `authentication mode changed` | Only one side has MD5 | Enable the same mode on both |
| Neighbor never forms | **Key string mismatch** | Re-enter the same `key-string` |
| Neighbor never forms | **Key ID mismatch** (1 vs 2) | Use the same key ID |
| Neighbor never forms | Key chain not bound to the interface | `ip authentication key-chain eigrp 1 <name>` |
| Neighbor never forms | Wrong AS in the authentication commands | Use the real AS (**1**) |
| Works, then fails later | Key lifetime expired | Check `accept-lifetime` / `send-lifetime` and the clock |
| `% Invalid input` | Typo or wrong config mode | Retype in the correct mode |

> 💡 Key chain **names** can differ between routers. Key **ID and string** must match.

---

## 🚀 Advanced Topics

### 🔄 Key Rotation with Lifetimes

Overlapping keys allow a **graceful rollover** without dropping the adjacency:

```bash
key chain khan
 key 1
  key-string cisco
  accept-lifetime 00:00:00 Jan 1 2026 00:00:00 Jan 1 2027
  send-lifetime   00:00:00 Jan 1 2026 00:00:00 Jan 1 2027
 key 2
  key-string NewStr0ngKey!
  accept-lifetime 00:00:00 Dec 1 2026 infinite
  send-lifetime   00:00:00 Jan 1 2027 infinite
```

> ⏰ Lifetimes depend on correct device time — use **NTP**.

### 🔒 Hardening Ideas

- Use a **long, complex key string** — `cisco` is fine for a lab but weak in production
- `service password-encryption` can obfuscate key strings in the config
- Add `passive-interface` on LAN-facing interfaces so EIGRP isn't sent to PCs
- On newer IOS, **EIGRP named mode** supports stronger **HMAC-SHA-256** authentication

---

## 🎓 Key Takeaways

1. 🔸 EIGRP without authentication trusts **any** router in the AS
2. 🔸 MD5 needs **three things**: key chain → mode `md5` → key chain bound to the **interface**
3. 🔸 Authentication is configured **per interface**, not globally
4. 🔸 **Key ID and key string must match**; the key chain *name* is local
5. 🔸 MD5 on **one side only** drops the adjacency (`authentication mode changed`)
6. 🔸 Matching config on the other side brings it back (`new adjacency`)
7. 🔸 MD5 gives **authenticity + integrity**, **not confidentiality**
8. 🔸 Always verify with `show ip eigrp neighbors` and a ping after securing a link

---

## 🧪 Challenges

1. Change the key string on **R2 only** — does the neighbor drop? What does `debug eigrp packets` show?
2. Use **different key IDs** (1 on R1, 2 on R2) — what happens?
3. Rename the key chain on R2 (e.g., `secure`) — does the adjacency survive?
4. Add a **second key** with lifetimes and test a smooth rollover
5. Attach a third router **without** authentication — can it become a neighbor?

---

## 📁 Repository Structure

```
📦 EIGRP-MD5-Authentication
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ 01-r1-key-chain.png
 ┃ ┣ 🖼️ 02-r1-md5-adjacency-down.png
 ┃ ┣ 🖼️ 03-r2-md5-adjacency-up.png
 ┃ ┗ 🖼️ 04-r1-running-config.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-MD5-Authentication.pkt
```

---

## 🔗 EIGRP Lab Series

| # | Lab |
|:-:|:----|
| 1 | EIGRP Successor – Single Path |
| 2 | EIGRP Successor – 2 Paths (Equal-Cost Load Balancing) |
| 3 | EIGRP Unequal-Cost Load Balancing & Variance |
| 4 | **EIGRP MD5 Authentication** ◄── *you are here* |
| 5 | EIGRP Metric Calculation |

`EIGRP` · `MD5` · `Authentication` · `Key Chain` · `Routing Security` · `Neighbor Adjacency` · `Network Hardening`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, please star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
