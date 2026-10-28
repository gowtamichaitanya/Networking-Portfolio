# 🌐 Networking Portfolio

Hands-on networking labs documenting my journey through **CCNA → CCNP Service Provider → CCNP Enterprise** — starting from the fundamentals and building up to larger, real-world style designs.

I work as a network engineer in the telecom sector (8+ years) with hands-on experience in routing, switching, NAT, IPv6, WLAN, OSPF, ACLs, IP planning and firewalls, across Cisco, Huawei, Nokia and Tejas equipment. Every lab here includes a topology, full device configs, verification output and what I learned — so anyone can rebuild it.

> All labs use **lab-only IP addressing**. No production addresses, hostnames or credentials are published.

---

## 🧭 Learning Path — Small to Big

Labs grow in size and complexity: each one builds on the previous.

### 🟢 Level 1 — Foundations (CCNA)
| # | Lab | Topics | Status |
|---|-----|--------|--------|
| 01 | [Inter-VLAN Routing — Router-on-a-Stick](./01-router-on-a-stick/) | VLANs, 802.1Q trunk, dot1Q sub-interfaces | ✅ Done |
| 02 | [Inter-VLAN Routing — SVIs on a Layer 3 Switch](./02-inter-vlan-svi/) | SVIs, `ip routing`, transit VLAN, ROAS vs SVI | ✅ Done |
| 03 | Trunks, Native VLAN & DTP | 802.1Q native VLAN, mismatch, DTP | 🔜 Next |
| 04 | Static & Default Routing | Static routes, floating statics | 📋 Planned |
| 05 | DHCP & NAT/PAT | DHCP server/relay, static NAT, PAT | 📋 Planned |

### 🟡 Level 2 — Building Networks (CCNA)
| # | Lab | Topics | Status |
|---|-----|--------|--------|
| — | OSPF single & multi-area | OSPFv2, DR/BDR, areas | 📋 Planned |
| — | STP & EtherChannel | Rapid PVST+, LACP | 📋 Planned |
| — | ACLs & Device Security | Standard/extended ACLs, SSH, port security | 📋 Planned |
| — | Campus Network Capstone | All Level 1–2 topics combined | 📋 Planned |

### 🔧 Physical Lab — real Cisco hardware
| # | Lab | Topics | Status |
|---|-----|--------|--------|
| P01 | [Cisco Boot Process & Config Register](./P01-boot-and-config-register/) | Config register, ROMMON, password recovery, TFTP backup, AutoInstall — hybrid 1841 + GNS3 | 🚧 In progress |

### 🔴 Level 3 — Service Provider (CCNP SP)
| # | Lab | Topics | Status |
|---|-----|--------|--------|
| — | Multi-VLAN NAT Gateway (telecom exchange design) | Router-on-a-stick, mixed static NAT/PAT | 📋 Planned |
| — | IS-IS & BGP | IGP/EGP, iBGP/eBGP | 📋 Planned |
| — | MPLS L3VPN | LDP, VRFs, MP-BGP | 📋 Planned |

---

## 🛠️ Lab Environment

| Component | Details |
|-----------|---------|
| Emulator | GNS3 on Fedora Linux 44 |
| Hypervisor | QEMU |
| Router image | Cisco vIOS 15.9 (`vios-adventerprisek9-m.spa.159-3.m6`) |
| Switch image | Cisco vIOS-L2 (`vios_l2-adventerprisek9-m.ssa.high_iron_20200929`) |
| End hosts | VPCS |
| Physical lab | Cisco routers & switches, ME 3400, WLC/APs, IP phones |

> Cisco images are licensed software and are **not** included in this repo.

---

## 📚 Certification Roadmap

- [ ] **CCNA 200-301** — in progress
- [ ] **CCNP Service Provider**
- [ ] **CCNP Enterprise**

---

## 📁 Repository Structure

```
networking-portfolio/
├── README.md                     ← you are here
├── 01-router-on-a-stick/
│   ├── README.md                 ← lab write-up
│   ├── configs/                  ← device configs (R1, SW1, PCs)
│   ├── diagrams/                 ← topology screenshot
│   ├── verification/             ← show command outputs
│   └── gns3/                     ← GNS3 project file
├── 02-inter-vlan-svi/            ← same layout for every lab
├── P01-boot-and-config-register/ ← physical lab (console logs instead of GNS3 files)
└── templates/
    └── LAB_README_TEMPLATE.md    ← template for new labs
```

---

## 📬 Connect

- GitHub: [@gowtamichaitanya](https://github.com/gowtamichaitanya)
- LinkedIn: _add link_
- YouTube: **Networking Made Easy** _(coming soon)_
