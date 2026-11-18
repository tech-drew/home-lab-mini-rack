# Network Design V2: Simplified 10G Edge & Platform Engineering Lab

This document outlines a redesigned **secure, high-performance VLAN and switching architecture** replacing the previous MikroTik RB5009 and Wyse 5070 thin-client stack. The primary goal is to **minimize underlying switching and routing maintenance**—reserving the complexity budget for Kubernetes, GitOps, and platform engineering automation—while preserving **high-speed Jumbo Frame (MTU 9000) East-West storage and backup throughput**.

> **Architectural Note (Design Evolution):** This design is currently a theoretical target state. Previously, MikroTik hardware served as an ideal hands-on platform for mastering core networking fundamentals. As my focus shifts toward Platform Engineering—introducing automated firewall policies, Infrastructure-as-Code (IaC), and GitOps pipelines—standardizing on a streamlined UniFi Layer 2 fabric is a deliberate trade-off to eliminate hardware quirks and reduce operational overhead. All of the routing and segmentation fundamentals gained on MikroTik will continue to be directly applied when configuring and automating the inline firewall VM (Firewalla, Sophos, or Palo Alto).

---

## Core Architectural Philosophy

* **Zero-Maintenance Security Edge:** A dedicated standalone **Lenovo M90q Tiny (i5-10500T, 32GB RAM)** runs **Proxmox VE** as its bare-metal operating system. All edge and infrastructure workloads are virtualized on top—including the **Firewalla Crystal SE** Firewall/Router VM alongside isolated LXC containers for the **UniFi Network Controller**, **Technitium DNS**, **Nginx Reverse Proxy**, and **PXE Bare-Metal Provisioning**.
* **10G Layer 2 Storage & Backup Backplane:** Heavy East-West traffic (ZFS+iSCSI, Ceph/NFS storage, ZFS replication, and Proxmox Backup Server jobs) stays strictly at **Layer 2** across the **UniFi USW-Aggregation** and **UniFi Pro Max 16 PoE** switches with **MTU 9000**, avoiding router CPU bottlenecks and MTU fragmentation.
* **Single-SSID Wireless Segmentation:** Consumer and IoT devices are segmented via **UniFi Private Pre-Shared Keys (PPSK)** on a single SSID mapped directly to wired VLANs, eliminating the need for 802.1X/RADIUS infrastructure. Transitioning wireless to UniFi also avoids the client-roaming headaches common on MikroTik wireless—while MikroTik excels at routing, UniFi provides significantly smoother, lower-maintenance Wi-Fi roaming out of the box.
* **Out-of-Band (OOB) Provisioning via Native VLAN 1:** The Lenovo M90q Tiny nodes feature **Intel vPro (AMT)** for hardware-level power control, remote KVM, and bare-metal PXE imaging. Because Intel AMT and bare-metal PXE boot firmware require untagged traffic on the **Native VLAN**, **VLAN 1** is dedicated strictly to vPro OOB management and PXE provisioning (`netboot.xyz` / Matchbox).

---

## Hardware Topology & Port Map

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/204b966c-90eb-42d0-b3ad-84a3e1bd29a9" />

```text
[ISP Modem] ──(1GbE RJ45 WAN)──> [1x Lenovo M90q Edge Host (Proxmox VE)]
                                    ├─ VM 100: Firewalla Crystal SE (IDS/IPS Router)
                                    ├─ LXC 101: UniFi Network Controller (VLAN 99)
                                    ├─ LXC 102: Technitium DNS (VLAN 10)
                                    ├─ LXC 103: Nginx Reverse Proxy (VLAN 10)
                                    └─ LXC 104: PXE Imaging Server (VLAN 1 Native)
                                         │
                             (10G Fiber Trunk: VLANs 1, 10, 20, 30, 40, 99)
                                         ▼
                           [UniFi USW-Aggregation (8x 10G SFP+)]
                             ├─ Ports 1–6 ──> [6x Lenovo M90q Tiny Nodes (K8s + Proxmox HA)]
                             ├─ Port 7    ──> (10G SFP+ DAC Uplink) ──> [UniFi Pro Max 16 PoE]
                             └─ Port 8    ──> (10G Fiber Uplink)   ──> [M90q Edge Host]

                           [UniFi Pro Max 16 PoE]
                             ├─ SFP+ Port 1 ──> Uplink from USW-Aggregation
                             ├─ SFP+ Port 2 ──> [TrueNAS Storage Server (10G SFP+)]
                             ├─ 2.5GbE Port ──> [Proxmox Backup Server (PBS)]
                             ├─ 1GbE Ports  ──> [6x M90q Onboard RJ45 (Intel vPro OOB / PXE)]
                             └─ PoE+ Ports  ──> [UniFi Wi-Fi Access Points (PPSK)]
```

---

## VLAN & Mixed-MTU Architecture

By keeping **VLAN 50 (Backup)** and **VLAN 60 (Storage)** as pure Layer 2 switch VLANs with no Layer 3 gateway on the Firewalla, high-throughput storage and backup flows remain air-gapped from user VLANs and operate at native **MTU 9000** without incurring router fragmentation overhead.

| VLAN ID | Name | Subnet | Gateway / DHCP | MTU | Purpose & Traffic Flow |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1 (Native)** | PROVISION / OOB | `10.100.1.0/24` | Firewalla (PXE DHCP + Option 66/67) | **1500** | **Untagged Native VLAN.** Intel vPro (AMT) OOB management and bare-metal PXE imaging (`netboot.xyz` / Matchbox) for M90q nodes. |
| **10** | SERVERS | `10.100.10.0/24` | Firewalla (Static / DHCP) | **1500** | Kubernetes control/worker VMs, Technitium DNS, Nginx Reverse Proxy. |
| **20** | TRUSTED | `10.100.20.0/24` | Firewalla (DHCP + DNS) | **1500** | Admin workstations, laptops, phones. Permitted into VLAN 1, 10, & 99. |
| **30** | GUEST | `10.100.30.0/24` | Firewalla (DHCP + DNS) | **1500** | Isolated visitors & unused ports; WAN access only (`Device Isolation`). |
| **40** | IOT | `10.100.40.0/24` | Firewalla (DHCP + DNS) | **1500** | Smart home devices, printers, cameras. Blocked from all internal VLANs. |
| **50** | BACKUP | `10.100.50.0/24` | **Switch-Only (No Gateway)** | **9000** | **Static Only.** Proxmox hosts ↔ Proxmox Backup Server (2.5G L2 backplane). |
| **60** | STORAGE | `10.100.60.0/24` | **Switch-Only (No Gateway)** | **9000** | **Static Only.** Proxmox hosts ↔ TrueNAS ZFS+iSCSI/Ceph (10G L2 backplane). |
| **99** | MGMT | `10.100.99.0/24` | Firewalla (Static / Staging DHCP) | **1500** | Firewalla, UniFi switches/APs, UniFi Controller, Proxmox hypervisor Web UIs. |

---

## Host Interface Isolation & Storage Sub-Interfaces

### 1. Edge Host Interface Isolation (M90q #7)
The dedicated edge M90q Tiny uses Linux bridges in Proxmox VE to keep WAN traffic isolated while sharing the 10G fiber uplink across Firewalla and infrastructure containers:

* **`vmbr1` (WAN Bridge — Onboard 1GbE RJ45):** No IP address assigned on the Proxmox host. Attached exclusively to `Firewalla Crystal SE` (`net1`) to air-gap the hypervisor from the raw ISP modem connection.
* **`vmbr0` (VLAN-Aware LAN Trunk — Mellanox 10G SFP+):** Carries untagged VLAN 1 and tagged VLANs into `USW-Aggregation` Port 8 with `VirtIO Multiqueue = 4` enabled on the Firewalla VM (`net0`).
* **Proxmox Host Management (`vmbr0.99`):** Static IP `10.100.99.2/24` on `VLAN 99`.
* **PXE Imaging Container (`LXC 104`):** Attached untagged to `vmbr0` (`VLAN 1 Native`) so bare-metal PXE DHCP broadcasts stay purely Layer 2 without requiring DHCP relay across subnets.

### 2. Compute Cluster Nodes (M90q #1–6)
* **Onboard 1GbE RJ45 (`VLAN 1 Native`):** Connected to `Pro Max 16 PoE` access ports configured with **VLAN 1 Native** to support hardware-level **Intel vPro (AMT)** out-of-band power/KVM control and bare-metal PXE boot.
* **Mellanox 10G SFP+ (`vmbr0` Trunk):** Connected to `USW-Aggregation` Ports 1–6 with `vmbr0.50` (`MTU 9000`) and `vmbr0.60` (`MTU 9000`) host sub-interfaces for direct Layer 2 TrueNAS ZFS+iSCSI storage and PBS backups.

---

## Baseline Firewall Policy (Firewalla)

1. **Out-of-Band & Provisioning Lock:** Block `All Local Networks` → `PROVISION / OOB (VLAN 1)` (Allow `TRUSTED (VLAN 20)` only for vPro/MeshCentral access).
2. **Management Protection:** Block `All Local Networks` → `MGMT (VLAN 99)` (Allow `TRUSTED (VLAN 20)`).
3. **Workload Isolation:** Block `SERVERS (VLAN 10)` → `TRUSTED (VLAN 20)` (Unidirectional block; established return traffic from `TRUSTED` sessions is automatically permitted).
4. **IoT Containment:** Block `IOT (VLAN 40)` → `All Local Networks` (Allow WAN only).
5. **Guest Isolation:** Enable native `Guest Network` isolation on `GUEST (VLAN 30)`.
6. **Storage & Backup Air-Gap:** Enforced at Layer 2 by omitting `VLAN 50` and `VLAN 60` interfaces on the Firewalla gateway.
