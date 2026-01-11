## Network Design Summary

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

Following the **CIS (Center for Internet Security) Critical Security Controls**, VLAN 1 is strictly disabled for all data and management paths. All trunk ports utilize a **Native Sink VLAN (999)** as their native PVID. This ensures that any untagged frames entering a trunk port are "sunk" into a non-routable VLAN, effectively neutralizing lateral movement and VLAN hopping exploits.

---

## Goals

* **Create an enterprise-style secure and scalable VLAN Design**
* **Follow Enterprise Networking and CyberSecurity Best Practices**
* **Learn how to be a better tech professional**

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name        | Subnet         | Purpose                             |
| ------- | ----------- | -------------- | ----------------------------------- |
| 10      | SERVERS     | 10.100.10.0/24 | VMs and containers only             |
| 20      | TRUSTED     | 10.100.20.0/24 | Admin devices, Laptops, Phones      |
| 30      | GUEST       | 10.100.30.0/24 | Guest Wi-Fi                         |
| 40      | IOT         | 10.100.40.0/24 | IoT devices, printers, cameras      |
| 50      | BACKUP      | 10.100.50.0/24 | Backup NAS (TrueNAS)                |
| 60      | STORAGE     | 10.100.60.0/24 | Proxmox Storage / Cluster Heartbeat |
| 99      | MGMT        | 10.100.99.0/24 | Router, Switches, APs, PVE Hosts    |
| 999     | NATIVE_SINK | None           | Dead-end for Untagged Trunk Traffic |

---

## VLAN Access Control Summary

**VLAN 10 (Servers)** hosts application workloads. Accessible only from Trusted (20) and MGMT (99).

**VLAN 20 (Trusted)** is the primary admin zone. Full access to Servers (10), Storage (60), and MGMT (99).

**VLAN 30 (Guest) and VLAN 40 (IoT)** are isolated with WAN access only. No internal lateral movement.

**VLAN 50 (Backup)** is restricted to authorized backup initiators within VLAN 10 (Servers).

**VLAN 60 (Storage)** is a high-priority, dedicated segment for **Proxmox HA storage, replication, and migration traffic**. Access is restricted to Proxmox host address lists.

**VLAN 99 (Management)** is for infrastructure hardware. Not reachable from Guest or IoT.

**VLAN 999 (NATIVE_SINK)** has no Layer 3 interface. It serves as a security catch-all for untagged traffic on trunk ports, ensuring unauthorized packets are discarded by the bridge.

---

## Physical Port Layout (RB5009)

| Port          | Device          | Mode                       | Native VLAN (PVID) |
| ------------- | --------------- | -------------------------- | ------------------ |
| ether1        | ISP Modem       | WAN                        | N/A                |
| ether2        | CAP ax AP       | Trunk (10, 20, 30, 40, 99) | 999 (NATIVE_SINK)  |
| ether3        | Spare/Admin     | Access                     | 99 (MGMT)          |
| ether4        | Admin/Wired PC  | Trunk (20, 60, 99)         | 999 (NATIVE_SINK)  |
| ether5–ether8 | Proxmox Cluster | Hybrid Trunk (10, 60)      | 99 (MGMT)          |

---

## MikroTik Administration: Winbox and Safe Mode

To manage an RB5009 effectively, two tools are essential: Winbox and Safe Mode. Together, they provide a "safety net" that allows you to perform complex network reconfigurations without the risk of a permanent lockout.

### What is Winbox?

Winbox is a native MikroTik management utility designed specifically for RouterOS. It offers a fast, robust, and visual interface for system administration.

* **MAC-Level Access:** Unlike a web browser, Winbox can connect to a router via its MAC address. This ensures you maintain control even if you misconfigure an IP address or break a VLAN setting.
* **Visual Workflow:** Its multi-window interface allows you to monitor logs, traffic graphs, and firewall rules simultaneously in real-time.
* **Device Discovery (Neighbors):** Winbox automatically scans your local network for MikroTik devices.

### What is Safe Mode?

Safe Mode acts as a stateful "Undo" button for your router’s configuration. When enabled, the router tracks every change made during that specific session in a temporary buffer.

* **Automatic Rollback:** If your management connection is interrupted, RouterOS rolls back all changes made during the Safe Mode session.
* **Prevents Lockouts:** Eliminates the need for physical access due to configuration errors.

### Safe Mode Workflow

1. **Enter Safe Mode** (`CTRL + X`)
2. **Apply Changes**
3. **Verify Access**
4. **Exit Safe Mode** to commit

---

## Key Configuration Snippets

### 1. Global Bridge Configuration

```routeros
/interface bridge
set bridge protocol-mode=none
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

1. **VLAN Hopping Mitigation:**
   Using PVID 999 as the **Native Sink** on trunk ports prevents double-tagging and VLAN hopping attacks.

2. **Storage Network Design (VLAN 60):**
   VLAN 60 is isolated to reduce contention and protect east–west storage traffic.
   **Optional:** Jumbo Frames (MTU 9000) *can* be enabled on this VLAN if all devices support it end-to-end. However, on a 1 GbE network, this typically yields only **minor (5–10%) efficiency gains** and **no noticeable real-world performance improvement**. Simplicity and consistency are often preferable.

3. **Proxmox Management Simplicity:**
   PVE hosts use untagged management access (PVID 99) to provide a reliable recovery path if Proxmox bridge or VLAN configuration becomes corrupted.

4. **No Layer 3 for Sink VLAN:**
   Never assign an IP address to the **NATIVE_SINK (999)** VLAN. It must remain unroutable and non-participatory in Layer 3.
