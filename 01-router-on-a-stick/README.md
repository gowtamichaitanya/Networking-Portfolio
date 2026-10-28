# Lab 01 — Inter-VLAN Routing with Router-on-a-Stick

![Level](https://img.shields.io/badge/level-beginner-brightgreen) ![Track](https://img.shields.io/badge/track-CCNA-blue) ![Platform](https://img.shields.io/badge/platform-GNS3-orange)

## 📌 Scenario

Two departments sit on the same switch but in different VLANs. VLANs keep their broadcast domains apart — which also means PC1 (VLAN 10) and PC2 (VLAN 20) **cannot talk to each other** without a Layer 3 device.

**Router-on-a-Stick (ROAS)** solves this with a single physical link: the switch sends both VLANs to the router over an **802.1Q trunk**, and the router uses one **sub-interface per VLAN** as each VLAN's default gateway.

**Objectives**
- Create VLANs 10 and 20 and assign access ports
- Build an 802.1Q trunk between SW1 and R1
- Configure dot1Q sub-interfaces on R1 as VLAN gateways
- Verify PC1 ↔ PC2 reachability through the router

---

## 🗺️ Topology

![Topology](./diagrams/topology.png)

```mermaid
flowchart TB
    R1["R1 (vIOS 15.9)<br/>Gi0/1.10 = 10.0.10.254<br/>Gi0/1.20 = 20.0.20.254"]
    SW1["SW1 (vIOS-L2)"]
    PC1["PC1 — VLAN 10<br/>10.0.10.1/24"]
    PC2["PC2 — VLAN 20<br/>20.0.20.1/24"]
    R1 == "Gi0/1 ⇄ Gi0/1<br/>802.1Q trunk (VLAN 10,20)" === SW1
    SW1 -- "Gi1/0 access VLAN 10" --- PC1
    SW1 -- "Gi2/0 access VLAN 20" --- PC2
```

---

## 🔢 Addressing Plan

| Device | Interface | IP / Mask | VLAN | Role |
|--------|-----------|-----------|------|------|
| R1 | Gi0/1.10 | 10.0.10.254/24 | 10 | VLAN 10 default gateway |
| R1 | Gi0/1.20 | 20.0.20.254/24 | 20 | VLAN 20 default gateway |
| SW1 | Gi0/1 | — | 10,20 (trunk) | Uplink to R1 |
| SW1 | Gi1/0 | — | 10 (access) | PC1 |
| SW1 | Gi2/0 | — | 20 (access) | PC2 |
| PC1 | e0 | 10.0.10.1/24 | 10 | GW 10.0.10.254 |
| PC2 | e0 | 20.0.20.1/24 | 20 | GW 20.0.20.254 |

---

## ⚙️ Configuration

Full configs: [`configs/`](./configs/)

### SW1 — VLANs, access ports, trunk
```cisco
vlan 10
vlan 20
!
interface GigabitEthernet1/0
 description vlan10-access
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet2/0
 description vlan20-access
 switchport mode access
 switchport access vlan 20
!
interface GigabitEthernet0/1
 description Trunk-UPLINK
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```
> VLANs are stored in `vlan.dat`, not in the running-config, which is why `vlan 10` / `vlan 20` don't appear in `configs/SW1.cfg`. Assigning an access port to a VLAN auto-creates it.

### R1 — sub-interfaces
```cisco
interface GigabitEthernet0/1
 no shutdown
!
interface GigabitEthernet0/1.10
 description vlan10-GW
 encapsulation dot1Q 10
 ip address 10.0.10.254 255.255.255.0
!
interface GigabitEthernet0/1.20
 description vlan20-GW
 encapsulation dot1Q 20
 ip address 20.0.20.254 255.255.255.0
```
> The physical interface carries **no IP address** — it only has to be up. Each sub-interface handles one VLAN tag.

### PCs (VPCS)
```
PC1> ip 10.0.10.1 10.0.10.254 24
PC2> ip 20.0.20.1 20.0.20.254 24
```

---

## 🔍 How It Works — PC1 pings PC2

1. PC1 sees 20.0.20.1 is **not** in its own subnet, so it sends the frame to its gateway's MAC (R1 Gi0/1.10).
2. SW1 receives it untagged on Gi1/0 (VLAN 10) and forwards it out the trunk **tagged VLAN 10**.
3. R1 receives the tag-10 frame on Gi0/1.10, strips the tag, and routes the packet — 20.0.20.0/24 is directly connected on Gi0/1.20.
4. R1 rewrites the Layer 2 header (new source MAC = R1, destination MAC = PC2) and sends it back down the **same physical link tagged VLAN 20**.
5. SW1 forwards it out Gi2/0 untagged to PC2. The reply takes the reverse path.

The packet crosses the trunk **twice** — once up, once down. That's the "stick", and it's also ROAS's main limitation: all inter-VLAN traffic shares one link's bandwidth.

---

## 🦈 Packet Capture — Following One Ping

I captured a ping from PC1 → PC2 at all three links with Wireshark to watch the frame change hop by hop.

| Capture point | 802.1Q tag | Src MAC | Dst MAC | TTL | Frame size |
|---------------|-----------|---------|---------|-----|------------|
| ① PC1 → SW1 (access) | — untagged | PC1 `00:50:79:66:68:00` | R1 `0c:fb:8e:7f:00:01` | 64 | 98 bytes |
| ② SW1 → R1 (trunk, going up) | **VLAN 10** | PC1 `00:50:79:66:68:00` | R1 `0c:fb:8e:7f:00:01` | 64 | 102 bytes |
| ③ R1 → SW1 (trunk, coming down) | **VLAN 20** | R1 `0c:fb:8e:7f:00:01` | PC2 `00:50:79:66:68:01` | **63** | 102 bytes |
| ④ SW1 → PC2 (access) | — untagged | R1 `0c:fb:8e:7f:00:01` | PC2 `00:50:79:66:68:01` | 63 | 98 bytes |

### ① Access link (PC1 side) — no VLAN tag
![PC1 access link](./captures/01-pc1-access-untagged.png)
*PC1 doesn't know VLANs exist: plain Ethernet II, Type `0x0800` (IPv4). Destination MAC is already R1 — PC1 is sending to its gateway.*

### ② Trunk, going up — tagged VLAN 10
![Trunk VLAN 10](./captures/02-trunk-vlan10.png)
*SW1 inserted an 802.1Q header with `ID: 10`. Ethernet Type is now `0x8100`, and the frame grew from 98 to 102 bytes — the 4-byte tag. MACs and TTL are unchanged: a switch doesn't touch Layer 3.*

### ③ Trunk, coming down — tagged VLAN 20
![Trunk VLAN 20](./captures/03-trunk-vlan20.png)
*Same ping (same ICMP id/seq), same physical link — but now `ID: 20`. R1 routed it: source MAC rewritten to R1, destination MAC is PC2, TTL decremented 64 → 63. The ARP exchanges for 10.0.10.254 and 20.0.20.254 are also visible at the top. Wireshark shows each request twice on this link (TTL 64 and TTL 63) — that's the packet crossing the trunk up and back down.*

### ④ Access link (PC2 side) — tag removed
![PC2 access link](./captures/04-pc2-access-untagged.png)
*SW1 stripped the tag before sending out the access port. PC2 receives a normal untagged frame with TTL 63 — proof it passed through exactly one router.*

### Key observations
- **Tagging happens only on the trunk.** Access ports send and receive untagged frames; the switch adds the tag on ingress to the trunk and removes it on egress to an access port.
- **The same packet crosses the trunk twice** with two different tags — the defining behaviour of router-on-a-stick.
- **Both sub-interfaces share one MAC** (`0c:fb:8e:7f:00:01`) — they're logical interfaces on the same physical port.
- **L2 header is rebuilt at the router, L3 is preserved.** IP addresses stay 10.0.10.1 → 20.0.20.1 end to end; only MACs, VLAN tag and TTL change.

📁 Capture files (open in Wireshark): [pc1-access.pcapng](./captures/pc1-access.pcapng) · [trunk.pcapng](./captures/trunk.pcapng) · [pc2-access.pcapng](./captures/pc2-access.pcapng)

---

## ✅ Verification

Command outputs go in [`verification/`](./verification/).

| # | Device | Command | What to look for |
|---|--------|---------|------------------|
| 1 | SW1 | `show vlan brief` | Gi1/0 in VLAN 10, Gi2/0 in VLAN 20 |
| 2 | SW1 | `show interfaces trunk` | Gi0/1 trunking, 802.1q, VLANs 10,20 allowed & forwarding |
| 3 | R1 | `show ip interface brief` | Gi0/1, Gi0/1.10, Gi0/1.20 up/up |
| 4 | R1 | `show vlans` | dot1Q sub-interfaces and their VLAN IDs |
| 5 | R1 | `show ip route connected` | 10.0.10.0/24 and 20.0.20.0/24 as `C` routes |
| 6 | PC1 | `ping 10.0.10.254` | Gateway reachable |
| 7 | PC1 | `ping 20.0.20.1` | Inter-VLAN reachability ✅ |
| 8 | PC1 | `trace 20.0.20.1` | First hop is 10.0.10.254 (traffic goes via R1) |

---

## 🧠 What I Learned

- VLANs separate broadcast domains — different VLANs **need Layer 3** to communicate.
- One physical router port can serve many VLANs using **802.1Q sub-interfaces**.
- The VLAN number in `encapsulation dot1Q <id>` must match the VLAN tag the switch sends — the sub-interface number (`.10`) is just a label, though matching them is good practice.
- On vIOS-L2, `switchport trunk encapsulation dot1q` must be set **before** `switchport mode trunk`.
- ROAS is simple and cheap, but the single trunk is a bottleneck and single point of failure — which is why the next lab uses SVIs on a Layer 3 switch.

---

## 📝 Notes

- `20.0.20.0/24` is public IP space. That's fine in an isolated lab, but real networks should use private ranges (RFC 1918: 10/8, 172.16/12, 192.168/16). Future labs will use private addressing.

---

## 🔁 Rebuild This Lab

1. GNS3 with **vIOS 15.9** (router) and **vIOS-L2** (switch) on QEMU, plus two VPCS.
2. Open `gns3/ROAS.gns3` (your image names must match) or build the topology from the diagram.
3. Paste `configs/R1.cfg` and `configs/SW1.cfg` into the consoles; load `PC1.vpc` / `PC2.vpc` into VPCS.
4. Run the verification commands.

---

➡️ Next: [Lab 02 — Inter-VLAN Routing with SVIs](../02-inter-vlan-svi/)
