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
