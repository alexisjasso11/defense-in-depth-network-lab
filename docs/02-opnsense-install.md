# 🔥 Phase 2 — OPNsense Firewall Install

![OPNsense](https://img.shields.io/badge/OPNsense-26.7-D94F00?style=for-the-badge&logo=opnsense&logoColor=white)
![FreeBSD](https://img.shields.io/badge/FreeBSD-Based-AB2B28?style=for-the-badge&logo=freebsd&logoColor=white)
![ZFS](https://img.shields.io/badge/Filesystem-ZFS-2A6EBB?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

[← Back to main page](../README.md)

## Lab Summary
Selected firewall hardware against defined requirements, installed OPNsense 26.7 bare-metal with verified installation media, resolved a failed first install, and completed initial configuration, patching, and a baseline backup.

## What I Built & Configured
- 📋 Evaluated about ten firewall hardware options against defined requirements
- 🔏 Verified the installer's SHA-256 checksum (supply-chain check)
- 💾 Installed OPNsense on ZFS for power-loss resilience and snapshots
- 🌐 Assigned WAN/LAN and resolved a subnet conflict with the home network
- 🧭 Configured Unbound as a recursive DNS resolver with DNSSEC
- 🩹 Patched to the latest firmware and took a baseline config backup

## 🔧 Step-by-Step Walkthrough

### Step 1 — Define Hardware Requirements

| Requirement | Reason |
|---|---|
| x86-64 CPU | OPNsense does not officially support ARM |
| 4+ Intel i226-V NICs | Best FreeBSD driver support; works with Suricata IPS mode; avoids early i225-V link issues |
| CPU headroom for inline IPS | Deep packet inspection is CPU-bound; target ~500 Mbps |
| Low power | Runs 24/7 |

**Selected:** Mrroute MR-M1 Plus — i3-N305 (8 cores), 16 GB onboard, 4x i226-V. Rejected options included 2013–2015 era CPUs that would bottleneck IPS, single-NIC/Realtek mini PCs, and used enterprise firewalls whose updates require paid support.

### Step 2 — Prepare and Verify Installation Media

| Setting | Value |
|---|---|
| Architecture | amd64 |
| Image type | vga (USB installer with monitor output) |
| Verification | SHA-256 of the `.img.bz2` matched the official page |
| Tools | 7-Zip (extract), Rufus (DD image mode) |

```
certutil -hashfile OPNsense-26.7-vga-amd64.img.bz2 SHA256
```

### Step 3 — Install OPNsense

| Setting | Value | Reason |
|---|---|---|
| Filesystem | ZFS | Copy-on-write protects against corruption on power loss; supports snapshots |
| Layout | Stripe | Single disk (mirror needs 2+, RAIDZ needs 3+) |
| Target | Samsung 128 GB NVMe | |

### Step 4 — Assign Interfaces and Fix a Subnet Conflict

| Interface | Port | Address |
|---|---|---|
| WAN | igc0 | DHCP from home router (temporary double NAT) |
| LAN | igc1 | 10.10.10.1/24, DHCP .100–.200 |

The default LAN address (192.168.1.1) overlapped the home network on the WAN side, which would break routing, so LAN was moved to `10.10.10.0/24`.

### Step 5 — Temporary WAN Path Through the Switch
```
interface GigabitEthernet1/0/3
 description OPNSENSE-WAN-TEMP
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
```
![Gi1/0/3 configured for OPNsense WAN](../images/4-%20TEMP-OPNSENSE%20CONF.png)

*Gi1/0/3 set as an access port in VLAN 100 with PortFast for the temporary OPNsense WAN link.*
### Step 6 — Setup Wizard

| Setting | Value | Reason |
|---|---|---|
| DNS | Unbound recursive, DNSSEC on | No reliance on ISP DNS |
| Timezone | America/Los_Angeles | Accurate log timestamps |
| WAN type | DHCP | Behind home router during build |
| Block private networks (WAN) | Temporarily off | WAN is on a private subnet; re-enable after bridge mode |
| Block bogons | On | Drops unassigned/reserved addresses |

![OPNsense Dashboard](../images/04-opnsense-dashboard.png)

### Step 7 — Patch and Back Up
- Updated to the latest firmware **before** any further configuration
- Downloaded a baseline config backup and stored it offline (never committed to this repo, since it contains password hashes)

## 🛠️ Troubleshooting & Challenges

### USB Installer Not Detected in BIOS
❌ **Problem:** The boot list only showed the internal SSD.

🔍 **Root Cause:** The compressed `.img.bz2` file was given to Rufus instead of the extracted raw image, so the USB wasn't bootable.

✅ **Fix:** Extracted the `.img` with 7-Zip, re-flashed in DD image mode, and confirmed Fast Boot and Secure Boot were disabled.

### Installation Failed Mid-Copy
❌ **Problem:** The ZFS install failed with `No such file or directory` errors.

🔍 **Root Cause:** The reused SSD still contained a previous Proxmox installation.

✅ **Fix:**
- Booted live mode → shell → `geom disk list` to identify the SSD (`nda0`)
- `gpart destroy -F nda0` returned `Device busy`, because the failed install still had the disk in use
- Rebooted cleanly, re-ran the command → `nda0 destroyed`
- Reinstalled successfully

### Booted Back Into the Installer
❌ **Problem:** After installing, the box returned to the live/installer login.

🔍 **Root Cause:** The USB was still plugged in and took boot priority.

✅ **Fix:** Removed the USB and rebooted; the system booted from the SSD.

## ✅ Phase Completion Summary

| Task | Status |
|---|---|
| Hardware selected against requirements | ✅ |
| Installer checksum verified | ✅ |
| OPNsense installed on ZFS | ✅ |
| WAN/LAN assigned, subnet conflict resolved | ✅ |
| Recursive DNS with DNSSEC | ✅ |
| Firmware patched | ✅ |
| Baseline config backup stored offline | ✅ |

## 📚 What I Learned
How to select hardware from requirements instead of specs alone, why NIC chipsets matter for a firewall, and how leftover partitions from a previous OS can break an installer. I also learned to plan addressing before cabling: catching the 192.168.1.0/24 overlap early prevented a confusing routing failure.
