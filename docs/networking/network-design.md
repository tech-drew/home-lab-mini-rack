## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

Following the **CIS (Center for Internet Security) Critical Security Controls**, VLAN 1 is strictly disabled for all data and management paths. All trunk ports utilize a **Native Sink VLAN (999)** as their native PVID. This ensures that any untagged frames entering a trunk port are "sunk" into a non-routable VLAN, effectively neutralizing lateral movement and VLAN hopping exploits.

---

## Goals

* **Create an enterprise-style secure and scalable VLAN Design:** 
* **Follow Enterprise Networking and CyberSecurity Best Practices:** 
* **Learn how to be a better tech professional:**

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name | Subnet | Purpose |
| --- | --- | --- | --- |
| 10 | SERVERS | 10.100.10.0/24 | VMs and containers only |
| 20 | TRUSTED | 10.100.20.0/24 | Admin devices, Laptops, Phones |
| 30 | GUEST | 10.100.30.0/24 | Guest Wi-Fi |
| 40 | IOT | 10.100.40.0/24 | IoT devices, printers, cameras |
| 50 | BACKUP | 10.100.50.0/24 | Backup NAS (TrueNAS) |
| 60 | STORAGE | 10.100.60.0/24 | Proxmox Storage / Cluster Heartbeat |
| 99 | MGMT | 10.100.99.0/24 | Router, Switches, APs, PVE Hosts |
| 999 | NATIVE_SINK | None | Dead-end for Untagged Trunk Traffic |

---

## VLAN Access Control Summary

**VLAN 10 (Servers)** hosts application workloads. Accessible only from Trusted (20) and MGMT (99).

**VLAN 20 (Trusted)** is the primary admin zone. Full access to Servers (10), Storage (60), and MGMT (99).

**VLAN 30 (Guest) and VLAN 40 (IoT)** are isolated with WAN access only. No internal lateral movement.

**VLAN 50 (Backup)** is restricted to authorized backup initiators within VLAN 10 (Servers).

**VLAN 60 (Storage)** is a high-priority, dedicated segment for **Proxmox HA storage/migration**. Access is restricted to Proxmox Host address lists. **Jumbo Frames (MTU 9000)** are enabled for high-throughput disk replication.

**VLAN 99 (Management)** is for infrastructure hardware. Not reachable from Guest or IoT.

**VLAN 999 (NATIVE_SINK)** has no Layer 3 interface. It serves as a security catch-all for untagged traffic on trunk ports, ensuring unauthorized packets are discarded by the bridge.

---

## Physical Port Layout (RB5009)

| Port | Device | Mode | Native VLAN (PVID) |
| --- | --- | --- | --- |
| ether1 | ISP Modem | WAN | N/A |
| ether2 | CAP ax AP | Trunk (10, 20, 30, 40, 99) | 999 (NATIVE_SINK) |
| ether3 | Spare/Admin | Access | 99 (MGMT) |
| ether4 | Admin/Wired PC | Trunk (20, 60, 99) | 999 (NATIVE_SINK) |
| ether5–ether8 | Proxmox Cluster | Hybrid Trunk (10, 60) | 99 (MGMT) |
---

## MikroTik Administration: Winbox and Safe Mode

To manage an RB5009 effectively, two tools are essential: Winbox and Safe Mode. Together, they provide a "safety net" that allows you to perform complex network reconfigurations without the risk of a permanent lockout.

### What is Winbox?

Winbox is a native MikroTik management utility designed specifically for RouterOS. It offers a fast, robust, and visual interface for system administration.

* **MAC-Level Access:** Unlike a web browser, Winbox can connect to a router via its MAC address. This ensures you maintain control even if you misconfigure an IP address or break a VLAN setting.
* **Visual Workflow:** Its multi-window interface allows you to monitor logs, traffic graphs, and firewall rules simultaneously in real-time.
* **Device Discovery (Neighbors):** Winbox automatically scans your local network for MikroTik devices. This makes initial setup and emergency recovery much simpler than searching for an unknown or misconfigured IP address.

### What is Safe Mode?

Safe Mode acts as a stateful "Undo" button for your router’s configuration. When enabled, the router tracks every change made during that specific session in a temporary buffer.

* **The Safety Trigger:** If your management connection is interrupted (e.g., you apply a firewall rule that blocks your own access), the router waits briefly. If the connection is not restored, it automatically rolls back every change made since Safe Mode was activated.
* **Peace of Mind:** This prevents the "Walk of Shame"—the need to physically access the router to perform a factory reset because of a simple configuration typo or logic error.

### Safe Mode Workflow

1. **Enter Safe Mode:** Before initiating any high-risk changes, click the **Safe Mode** button at the top left of Winbox (or press `CTRL + X`).
2. **Apply Changes:** Execute updates (VLAN IDs, bridge filtering, firewall).
3. **Verify Access:** Confirm connectivity to servers and the internet.
4. **Commit Changes:** Click the **Safe Mode** button again to toggle it off and save permanently.
---

## Key Configuration Snippets

### 1. Global Bridge & MTU Hardening

```routeros
# Set Jumbo Frames for the Bridge and Storage Ports
/interface bridge set bridge l2mtu=9000
/interface ethernet set [find where name~"ether[4-8]"] l2mtu=9000
```

### 2. Create the Native Sink VLAN

```routeros
/interface bridge vlan
add bridge=bridge vlan-ids=999 comment="NATIVE_SINK - Standard to drop untagged trunk traffic"
```

### 3. Port Assignment & Ingress Filtering

```routeros
/interface bridge port
set [find interface=ether2] pvid=999 frame-types=admit-only-vlan-tagged ingress-filtering=yes
set [find interface=ether3] pvid=99 ingress-filtering=yes
set [find interface=ether4] pvid=999 frame-types=admit-only-vlan-tagged ingress-filtering=yes
set [find interface=ether5] pvid=99 ingress-filtering=yes
set [find interface=ether6] pvid=99 ingress-filtering=yes
set [find interface=ether7] pvid=99 ingress-filtering=yes
set [find interface=ether8] pvid=99 ingress-filtering=yes
```

### 4. Enable Bridge VLAN Filtering

**Warning:** Only run this once all PVIDs and VLAN memberships are verified.

```routeros
/interface bridge set bridge vlan-filtering=yes
```

---

## Best Practices

1. **VLAN Hopping Mitigation:** By using PVID 999 as the **Native Sink** on `ether2` and `ether4`, "Double Tagging" attacks are neutralized at the switch chip level.
2. **Storage Performance (VLAN 60) & Jumbo Frames:** To maximize Proxmox HA performance, we use Jumbo Frames (MTU 9000).
  - Standard frames (1,500 bytes) require the CPU to process 6x more headers than a 9,000-byte Jumbo Frame.
  - This reduces CPU overhead and increases effective throughput during disk replication and VM migrations.
  - Note: Ensure your Proxmox Linux Bridges (vmbr0) and storage interfaces are also set to MTU 9000 to match the MikroTik.
3. **Proxmox Management:** PVE Hosts on `ether5-8` utilize PVID 99. This allows the host OS to be managed "untagged," providing a simpler recovery path if the Proxmox bridge configuration is corrupted.
4. **No Layer 3 for Sink:** Never assign an IP address to the NATIVE_SINK. The router must remain "deaf" to this VLAN.
