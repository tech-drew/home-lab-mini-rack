# MikroTik RB5009 – Enterprise-Style VLAN Segmentation Guide

---

## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

This design **explicitly separates infrastructure management from workloads**, following best practices for Proxmox, virtualization security, and network segmentation.

---

## Goals

* Separate **infrastructure management** from **server workloads**
* Secure Proxmox hypervisors from VM compromise
* Enforce security using **firewall rules**, not just VLANs
* Allow trusted Wi-Fi devices to access servers when required
* Isolate IoT, guest, and backup networks
* Scale cleanly in the future

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name    | Subnet         | Purpose                                  |
| ------: | ------- | -------------- | ---------------------------------------- |
|      10 | SERVERS | 10.100.10.0/24 | VMs and containers only              |
|      20 | TRUSTED | 10.100.20.0/24 | Laptops, phones (Wi-Fi)                  |
|      30 | GUEST   | 10.100.30.0/24 | Guest Wi-Fi                              |
|      40 | IOT     | 10.100.40.0/24 | IoT devices, printers, cameras           |
|      50 | BACKUP  | 10.100.50.0/24 | Backup NAS (TrueNAS)                     |
|      99 | MGMT    | 10.100.99.0/24 | Router, switches, APs, Proxmox hosts |

## IP Addressing Standards

* Gateways: `.1` of each subnet
* DHCP pools start at `.100`
* Static IPs for:

  * Proxmox hosts
  * Backup systems
  * Network infrastructure
  * Printers and APs

### Proxmox Host IPs (MGMT VLAN)

| Host  | IP Address   |
| ----- | ------------ |
| node1 | 10.100.99.11 |
| node2 | 10.100.99.12 |
| node3 | 10.100.99.13 |
| node4 | 10.100.99.14 |

---

## Physical Port Layout

| Port          | Device             | Mode                     |
| ------------- | ------------------ | ------------------------ |
| ether1        | ISP Modem          | WAN                      |
| ether2        | CAP ax AP          | **Trunk (tagged VLANs)** |
| ether3        | Spare              | Not assigned             |
| ether4        | TrueNAS Backup NAS | Access (VLAN 50)         |
| ether5–ether8 | Proxmox nodes      | **Trunk (VLANs 10, 99)** |

---

## Step 1 – Create the Bridge

```mikrotik
/interface bridge
add name=bridge vlan-filtering=yes
```

---

## Step 2 – Add Ports to the Bridge

```mikrotik
/interface bridge port
add bridge=bridge interface=ether2
add bridge=bridge interface=ether4 pvid=50
add bridge=bridge interface=ether5
add bridge=bridge interface=ether6
add bridge=bridge interface=ether7
add bridge=bridge interface=ether8
```

> **Note:** Proxmox ports have **no PVID** and operate as trunks.

---

## Step 3 – Define VLANs on the Bridge

```mikrotik
/interface bridge vlan
add bridge=bridge vlan-ids=10 tagged=bridge,ether5,ether6,ether7,ether8
add bridge=bridge vlan-ids=20 tagged=bridge,ether2
add bridge=bridge vlan-ids=30 tagged=bridge,ether2
add bridge=bridge vlan-ids=40 tagged=bridge,ether2
add bridge=bridge vlan-ids=50 tagged=bridge untagged=ether4
add bridge=bridge vlan-ids=99 tagged=bridge,ether5,ether6,ether7,ether8
```

---

## Step 4 – Create VLAN Interfaces

```mikrotik
/interface vlan
add interface=bridge name=vlan10 vlan-id=10
add interface=bridge name=vlan20 vlan-id=20
add interface=bridge name=vlan30 vlan-id=30
add interface=bridge name=vlan40 vlan-id=40
add interface=bridge name=vlan50 vlan-id=50
add interface=bridge name=vlan99 vlan-id=99
```

---

## Step 5 – Assign IP Addresses

```mikrotik
/ip address
add address=10.100.10.1/24 interface=vlan10
add address=10.100.20.1/24 interface=vlan20
add address=10.100.30.1/24 interface=vlan30
add address=10.100.40.1/24 interface=vlan40
add address=10.100.50.1/24 interface=vlan50
add address=10.100.99.1/24 interface=vlan99
```

---

## Step 6 – DHCP Servers

**No DHCP on SERVERS or MGMT VLANs**

```mikrotik
/ip pool
add name=pool20 ranges=10.100.20.100-10.100.20.200
add name=pool30 ranges=10.100.30.100-10.100.30.200
add name=pool40 ranges=10.100.40.100-10.100.40.200

/ip dhcp-server
add interface=vlan20 address-pool=pool20 disabled=no
add interface=vlan30 address-pool=pool30 disabled=no
add interface=vlan40 address-pool=pool40 disabled=no
```

---

## Proxmox Networking Model (Documented)

Each Proxmox node:

* Management IP on **VLAN 99**
* VLAN-aware bridge (`vmbr0`) with **no IP**
* VMs and containers tagged into:

  * VLAN 10 (SERVERS)
  * Additional VLANs only if explicitly required

This ensures:

* Hypervisors are isolated from workloads
* Compromised VMs cannot access management
* Clean firewall enforcement

---

## Firewall Policy Summary (MGMT Emphasis)

### Key Rules

* TRUSTED → MGMT: **Allowed**
* SERVERS → MGMT: **Denied**
* GUEST / IOT → MGMT: **Denied**
* Proxmox → BACKUP: **Allowed**
* Everything else → BACKUP: **Denied**

### Explicit MGMT Protection

```mikrotik
# Allow trusted devices to management
add chain=forward src-address=10.100.20.0/24 dst-address=10.100.99.0/24 action=accept

# Block all other access to management
add chain=forward dst-address=10.100.99.0/24 action=drop
```

---

## Best Practices (Updated)

* Hypervisors are **management infrastructure**
* Workloads never share a subnet with hypervisors
* Trunk ports for virtualization hosts
* Default deny between VLANs
* Static IPs for all infrastructure
* Backup network reachable **only** from hypervisors
* Document VLAN usage and IP assignments

---

## Final Notes

This design:

* Follows enterprise Proxmox deployment standards
* Strongly limits blast radius from VM compromise
* Keeps management isolated and auditable
* Scales cleanly as the lab grows
