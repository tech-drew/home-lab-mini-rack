# MikroTik RB5009 – Enterprise-Style VLAN Segmentation Guide

---

## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

This design **explicitly separates infrastructure management and storage backplanes from workloads**, following best practices for Proxmox clusters, virtualization security, and network segmentation.

---

## Goals

* Separate **infrastructure management** from **server workloads**.
* Implement a dedicated **Storage/HA Backplane** for Proxmox.
* Secure Proxmox hypervisors from VM compromise.
* Enforce security using **firewall rules**, not just VLANs.
* Allow trusted Wi-Fi devices to access servers and management when required.
* Isolate IoT, guest, and backup networks.

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name    | Subnet         | Purpose                               |
| ------: | ------- | -------------- | ------------------------------------ |
|      10 | SERVERS | 10.100.10.0/24 | VMs and containers only              |
|      20 | TRUSTED | 10.100.20.0/24 | Admin devices, Laptops, Phones       |
|      30 | GUEST   | 10.100.30.0/24 | Guest Wi-Fi                          |
|      40 | IOT     | 10.100.40.0/24 | IoT devices, printers, cameras       |
|      50 | BACKUP  | 10.100.50.0/24 | Backup NAS (TrueNAS)                 |
|      60 | STORAGE | 10.100.60.0/24 | Proxmox Storage / Cluster Heartbeat  |
|      99 | MGMT    | 10.100.99.0/24 | Router, Switches, APs, PVE Hosts     |

---

## VLAN Access Control Summary

* **VLAN 20 (Trusted):** Acts as the admin zone. It has full access to **VLAN 10 (Servers)**, **VLAN 60 (Storage)**, and **VLAN 99 (Management)** for administration.
* **VLAN 60 (Storage):** Strictly restricted. Only accessible by Proxmox Hosts and Admin devices. No internet access.
* **VLAN 10 (Servers):** Hosts application workloads. It has restricted access to **VLAN 50 (Backup)** but cannot reach Management or Storage.
* **VLAN 30/40 (Guest/IoT):** Isolated to WAN only. No internal lateral movement allowed.

---

## IP Addressing Standards

### Proxmox Infrastructure IPs

| Host       | MGMT IP (VLAN 99) | Storage IP (VLAN 60) |
| :--------- | :---------------- | :------------------ |
| pve-node-01| 10.100.99.11      | 10.100.60.11        |
| pve-node-02| 10.100.99.12      | 10.100.60.12        |
| pve-node-03| 10.100.99.13      | 10.100.60.13        |
| pve-node-04| 10.100.99.14      | 10.100.60.14        |

---

## Physical Port Layout (RB5009)

| Port          | Device             | Mode                        | Native VLAN (PVID) |
| :------------ | :----------------- | :-------------------------- | :----------------- |
| ether1        | ISP Modem          | WAN                         | N/A                |
| ether2        | CAP ax AP          | Trunk (10, 20, 30, 40, 99)  | 1 (Unused)         |
| ether3        | Spare/Admin        | Access                      | 99 (MGMT)          |
| ether4        | Admin/Wired PC     | Access                      | 20 (TRUSTED)       |
| ether5–ether8 | Proxmox Cluster    | Hybrid Trunk (10, 60)       | 99 (MGMT)          |

> **Note on Hybrid Trunks:** By setting PVID 99 on ether5-8, the Proxmox "untagged" traffic lands on the Management VLAN, while VM traffic (VLAN 10) and Storage traffic (VLAN 60) remain tagged.

---

## MikroTik Administration: Winbox and Safe Mode

To manage an RB5009 effectively, two tools are essential: Winbox and Safe Mode. Together, they provide a "safety net" that allows you to perform complex network reconfigurations without the risk of a permanent lockout.

### What is Winbox?

Winbox is a native MikroTik management utility designed specifically for RouterOS. It offers a fast, robust, and visual interface for system administration.

MAC-Level Access: Unlike a web browser, Winbox can connect to a router via its MAC address. This ensures you maintain control even if you misconfigure an IP address or break a VLAN setting.

Visual Workflow: Its multi-window interface allows you to monitor logs, traffic graphs, and firewall rules simultaneously in real-time.

Device Discovery (Neighbors): Winbox automatically scans your local network for MikroTik devices. This makes initial setup and emergency recovery much simpler than searching for an unknown or misconfigured IP address.

Winbox can be downloaded from: https://mikrotik.com/download/winbox

### What is Safe Mode?

Safe Mode acts as a stateful "Undo" button for your router’s configuration. When enabled, the router tracks every change made during that specific session in a temporary buffer.

The Safety Trigger: If your management connection is interrupted (e.g., you apply a firewall rule that blocks your own access), the router waits briefly. If the connection is not restored, it automatically rolls back every change made since Safe Mode was activated.

Peace of Mind: This prevents the "Walk of Shame"—the need to physically access the router to perform a factory reset because of a simple configuration typo or logic error.

### Why You Should Use Winbox and Safe mode

Reliability: Combining Winbox (via MAC connection) with Safe Mode is the gold standard for network engineers. It ensures that structural changes—such as enabling VLAN Filtering or modifying Firewall Chains—can be tested safely.

Operational Discipline: Using these tools demonstrates a professional "Change Management" mindset. Rather than guessing at commands, you are operating in a controlled environment where errors are non-destructive.

### Safe Mode Workflow

Follow these steps to ensure a safe configuration process:

Enter Safe Mode: Before initiating any high-risk changes, click the Safe Mode button at the top left of Winbox (or press CTRL + X in the Terminal).

Apply Changes: Execute your configuration updates, such as adjusting VLAN IDs, bridge settings, or firewall rules.

Verify Access: Confirm that you can still reach your servers, the internet, and the router’s management interface.

Commit Changes: Once you have verified that everything is functioning correctly, click the Safe Mode button again to toggle it off. This "commits" your changes permanently to the router's memory.

## Key Configuration Snippets

### Bridge Port & VLAN Tagging
```routeros
/interface bridge port
add bridge=bridge interface=ether3 pvid=99
add bridge=bridge interface=ether4 pvid=20
add bridge=bridge interface=ether5 pvid=99
add bridge=bridge interface=ether6 pvid=99
add bridge=bridge interface=ether7 pvid=99
add bridge=bridge interface=ether8 pvid=99

/interface bridge vlan
add bridge=bridge tagged=bridge,ether5,ether6,ether7,ether8 vlan-ids=10
add bridge=bridge tagged=bridge,ether2,ether5,ether6,ether7,ether8 untagged=ether3,ether4 vlan-ids=20
add bridge=bridge tagged=bridge,ether2,ether5,ether6,ether7,ether8 vlan-ids=60
add bridge=bridge tagged=bridge,ether2,ether5,ether6,ether7,ether8 vlan-ids=99

```

### Firewall Security Policy

The firewall is designed to allow management from the Trusted VLAN while dropping all unauthorized cross-VLAN traffic.

```routeros
# Allow Admin to reach Management & Storage
/ip firewall filter
add action=accept chain=forward comment="Allow Trusted to MGMT" dst-address=10.100.99.0/24 src-address=10.100.20.0/24
add action=accept chain=forward comment="Allow Trusted to Storage" dst-address=10.100.60.0/24 src-address=10.100.20.0/24

# Infrastructure Isolation
add action=drop chain=forward comment="Isolate MGMT" dst-address=10.100.99.0/24
add action=drop chain=forward comment="Isolate Storage" dst-address=10.100.60.0/24
add action=drop chain=forward comment="Drop IoT/Guest to Trusted" in-interface-list=LAN out-interface=vlan20

```

---

## Best Practices

1. **Corosync Stability:** Use VLAN 60 for Proxmox Cluster communication to ensure high priority and low latency.
2. **Management Access:** Always use WinBox via the Trusted VLAN or the dedicated physical MGMT ports (ether3, 5-8).
3. **Backup Isolation:** Ensure the Backup NAS (VLAN 50) is only reachable by the specific IP addresses of the Proxmox hosts via Address Lists.


---
