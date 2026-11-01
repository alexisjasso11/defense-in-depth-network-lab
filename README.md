# 🛡️ Defense-in-Depth Network Lab — Cisco Catalyst / OPNsense / Suricata

![Cisco](https://img.shields.io/badge/Cisco-Catalyst_9200CX-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![OPNsense](https://img.shields.io/badge/OPNsense-26.7-D94F00?style=for-the-badge&logo=opnsense&logoColor=white)
![Suricata](https://img.shields.io/badge/Suricata-IDS%2FIPS-EF3B2D?style=for-the-badge)
![Tailscale](https://img.shields.io/badge/Tailscale-Remote_Access-242424?style=for-the-badge&logo=tailscale&logoColor=white)
![Nmap](https://img.shields.io/badge/Nmap-Validation-4682B4?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-In_Progress-yellow?style=for-the-badge)

## Lab Summary
Designing and building a segmented home network the way a small enterprise would: a Cisco Catalyst access switch, an OPNsense stateful firewall with Suricata IDS/IPS, isolated VLANs, and secure remote access through Tailscale. Every security control is **validated with documented Nmap testing**, following a design → implement → validate → document workflow.

## 📊 Project Phases

| Phase | Description | Status | Documentation |
|---|---|---|---|
| 1 | Switch baseline: access hardening, SSH, management VLAN | ✅ Complete | [View](docs/01-switch-baseline.md) |
| 2 | OPNsense install, initial config, patching | ✅ Complete | [View](docs/02-opnsense-install.md) |
| 3 | Firewall hardening: 2FA, management-only GUI | ⏳ Next | — |
| 4 | VLAN segmentation + 802.1Q trunk | ⏳ Planned | — |
| 5 | Per-VLAN firewall rules | ⏳ Planned | — |
| 6 | Layer 2 security: DHCP snooping, DAI, port security | ⏳ Planned | — |
| 7 | Tailscale subnet router with ACLs | ⏳ Planned | — |
| 8 | Suricata IDS → IPS with performance testing | ⏳ Planned | — |
| 9 | Segmentation validation with Nmap | ⏳ Planned | — |
| 10 | Central logging + Wazuh SIEM | ⏳ Planned | — |

## 🗺️ Network Architecture

**Target design**
```
Internet
   │
ISP modem (bridge mode)
   │
OPNsense firewall ── routing, stateful firewall, NAT, DHCP, DNS, IDS/IPS, Tailscale
   │  802.1Q trunk
LAB-SW1 (Cisco Catalyst)
   ├── Access ports → wired clients, Proxmox host
   └── Trunk → VLAN-aware access point (SSIDs mapped to VLANs)
```

| VLAN | Name | Purpose | Policy |
|---|---|---|---|
| 100 | MGMT | Switch, firewall, AP management | Admin devices only |
| 10 | HOME | Trusted clients | Internet + limited Lab access |
| 20 | GUEST | Visitor Wi-Fi | Internet only |
| 30 | IOT | Smart devices | Internet only, no inbound initiation |
| 40 | LAB | Proxmox / Active Directory lab | Isolated by default |

## Hardware Used

| Role | Device | Key Specs |
|---|---|---|
| Access Switch | Cisco Catalyst C9200CX-12P-2X2G | 12x 1G PoE+, 2x 1G SFP, 2x 10G SFP+, IOS-XE 17.9 |
| Firewall | Mrroute MR-M1 Plus | Intel i3-N305 (8C/8T), 16 GB RAM, 4x Intel i226-V 2.5GbE |
| Firewall Storage | Samsung PM991 | 128 GB M.2 2242 NVMe |

## Software & Tools
- **Cisco IOS-XE 17.9** — switch operating system
- **OPNsense 26.7** — firewall/router platform (FreeBSD-based)
- **Suricata** — intrusion detection/prevention
- **Tailscale** — WireGuard-based remote access
- **Nmap** — segmentation and vulnerability validation
- **MobaXterm** — serial console and SSH client
- **Rufus** — installer media preparation

## 🎯 Skills Demonstrated
- 🔐 Network device hardening (Cisco IOS-XE, OPNsense)
- 🧱 Network segmentation and VLAN design
- 🔥 Stateful firewall policy design
- 🕵️ IDS/IPS deployment and performance tuning
- 🧪 Security control validation and findings documentation
- 🛠️ Structured troubleshooting (problem → root cause → fix)
- 📋 Hardware evaluation against defined requirements

## 🧠 Key Design Decisions
**Why route VLANs through the firewall instead of the switch?**
The Catalyst can route, but its ACLs are stateless and it can't do NAT, per-VLAN DHCP with DNS filtering, IDS/IPS, or VPN. Routing through OPNsense means every packet crossing a VLAN boundary is inspected by a stateful firewall.

**Why this firewall hardware?**
I defined requirements first (x86, 4+ Intel i226-V NICs, CPU headroom for inline IPS at ~500 Mbps, low 24/7 power) and evaluated about ten options. The 8-core i3-N305 with 16 GB onboard won on both performance and total cost once RAM prices were factored in.

**Why no open ports for remote access?**
Tailscale acts as a subnet router advertising only the management subnet, so nothing is exposed on the WAN.

## 🔒 Repository Security
- No password hashes, secrets, or firewall config backups are committed
- Only private (RFC 1918) addresses appear in documentation
- Screenshots are reviewed for typed passwords and serial numbers before upload
- All scanning is performed only on networks I own
