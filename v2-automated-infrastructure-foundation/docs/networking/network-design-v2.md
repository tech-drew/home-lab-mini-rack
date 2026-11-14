# Home Lab Network Architecture: UniFi 10GbE Fiber + Inline Firewall

## Design Overview
This architecture replaces the existing MikroTik equipment with a **100% fanless (0 dBA), low-power (~45W–55W) UniFi Layer 2 switching and Wi-Fi 7 stack** paired with an inline **Sophos or Palo Alto VM firewall** on a Lenovo M90Q Tiny.

* **Zero Data Backflow & Inline Security:** All UniFi VLANs are configured in **Third-Party Gateway (VLAN-only)** mode. Neither the `USW-Aggregation` switches nor the `USW-Pro-Max-16-PoE` perform Layer 3 routing, forcing all inter-VLAN traffic across 10GbE fiber trunks through the firewall VM's 802.1Q sub-interfaces.
* **Physical WAN/LAN Separation:** The ISP modem connects directly to the M90Q firewall host's built-in 1GbE RJ45 port (`vmbr0`), keeping untrusted WAN traffic completely off the switches.
* **Storage & Compute Performance:** Dual fanless `USW-Aggregation` switches provide 10GbE SFP+ fiber trunks (with 9000 MTU jumbo frames) for all six Cluster 2 nodes, TrueNAS, Proxmox Backup Server (PBS), and the firewall LAN trunk—with 4 open SFP+ ports for future LACP bonding.
* **Isolated PXE & Intel vPro Plane:** The `USW-Pro-Max-16-PoE` handles Cluster 1, Wi-Fi 7 PoE, and the six Cluster 2 built-in 1GbE NICs (configured with UniFi Port Isolation so provisioning/vPro ports cannot talk peer-to-peer).

**Note:** This design is currently theoretical. Previously, MikroTik hardware served as a fantastic hands-on tool for learning networking fundamentals. However, as I transition toward Platform Engineering, I will be introducing significant complexity through firewall policies, infrastructure automation, and GitOps workflows. Standardizing on a streamlined Layer 2 fabric is a strategic trade-off to eliminate hardware quirks and reduce overall operational overhead. All of the networking skills gained from MikroTik will still be actively applied when configuring and automating the inline Sophos or Palo Alto firewall VM.

---

## High-Level Wiring Topology

* **ISP Modem → M90Q Firewall Host (Built-in 1GbE RJ45 Port):**
  * Dedicated physical WAN bridge (`vmbr0`) passed directly into the Sophos/Palo Alto VM's WAN interface.
* **M90Q Firewall Host (ConnectX-4 SFP+ Port) → `USW-Aggregation` #2 (1x 10GBASE-SR Fiber Trunk):**
  * Dedicated internal LAN trunk (`vmbr1`, VLAN-aware) carrying all internal VLANs into the firewall's 802.1Q sub-interfaces.
* **`USW-Aggregation` #2 (Core Storage & Firewall — 4 of 8 SFP+ Ports Used):**
  * **Port 1:** M90Q Firewall VM (10GBASE-SR trunk — all internal VLANs).
  * **Port 2:** TrueNAS Storage Server (10GBASE-SR trunk — `Storage`, `Management` VLANs, 9000 MTU).
  * **Port 3:** Proxmox Backup Server (10GBASE-SR trunk — `Storage`, `Management` VLANs, 9000 MTU).
  * **Port 4:** Inter-switch 10GBASE-SR fiber trunk ↔ **`USW-Aggregation` #1**.
  * *(Ports 5–8 remain open for future dual-NIC LACP bonding on TrueNAS/PBS).*
* **`USW-Aggregation` #1 (Cluster 2 Compute Core — 8 of 8 SFP+ Ports Used):**
  * **Ports 1–6:** 6x M90Q Cluster 2 nodes (10GBASE-SR trunks — `Compute`, `Storage`, `Management` VLANs).
  * **Port 7:** Inter-switch 10GBASE-SR fiber trunk ↔ **`USW-Aggregation` #2**.
  * **Port 8:** Inter-switch 10GBASE-SR fiber trunk ↔ **`USW-Pro-Max-16-PoE`**.
* **`USW-Pro-Max-16-PoE` (1G/2.5G Access & PoE Layer):**
  * **SFP+ Port 1:** Inter-switch 10GBASE-SR fiber trunk ↔ **`USW-Aggregation` #1**.
  * **SFP+ Port 2:** Open spare 10G SFP+ port.
  * **6x 1GbE Ports:** 6x M90Q Cluster 2 built-in 1GbE NICs (dedicated `PXE / Intel vPro` access ports with Port Isolation enabled).
  * **4x 1GbE Ports:** 4x Dell Wyse 5070 nodes (Cluster 1 existing services + PXE server).
  * **1x 2.5GbE PoE++ Port:** `U7 Pro` Wi-Fi 7 AP (trunks wireless SSID VLANs).
  * **Remaining 5x Copper Ports:** TrueNAS/PBS out-of-band management or local workstation.

---

## Bill of Materials & Rough Cost

| Component | Model / Specification | Qty | Unit Est. | Subtotal Est. | Notes |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **10GbE Core Switches** | Ubiquiti `USW-Aggregation` (8x 10G SFP+) | 2 | ~$270 | ~$540 | Fanless (0 dBA), ~12–15W each |
| **1G/2.5G PoE Switch** | Ubiquiti `USW-Pro-Max-16-PoE` | 1 | ~$400 | ~$400 | Fanless (0 dBA), 12x 1G + 4x 2.5G + 2x SFP+ |
| **Wireless AP** | Ubiquiti `U7 Pro` (Wi-Fi 7) | 1 | ~$190 | ~$190 | Powered via 2.5GbE PoE++ port |
| **10G Fiber Optics** | 10GBASE-SR SFP+ 850nm Transceivers | 22 | ~$14 | ~$310 | 11 links total (Ubiquiti/Generic coded) |
| **Fiber Patch Cables** | OM3/OM4 Duplex LC-LC Fiber (0.5m–3m) | 11 | ~$7 | ~$77 | Thin, flexible cables for clean routing |
| **Cat6 Patch Cables** | Slimrun Cat6 RJ45 (0.5ft–3ft) | 13 | ~$3 | ~$39 | WAN, vPro/PXE, Cluster 1, AP, OOB |
| **UniFi Controller** | UniFi Network Application (Docker/LXC) | 1 | $0 | $0 | Self-hosted on Cluster 1 |
| **Estimated Total** | | | | **~$1,556** | *(Before taxes/shipping)* |
