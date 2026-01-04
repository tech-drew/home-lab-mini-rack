# MikroTik RB5009 – Enterprise-Style VLAN Segmentation Guide

---

## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

This design **explicitly separates infrastructure management from workloads**, following best practices for Proxmox, virtualization security, and network segmentation. It enforces isolation between VLANs and uses firewall rules to restrict unauthorized access.

---

## Goals

* Separate **infrastructure management** from **server workloads**.
* Secure Proxmox hypervisors from VM compromise.
* Enforce security using **firewall rules**, not just VLANs.
* Allow trusted Wi-Fi devices to access servers when required.
* Isolate IoT, guest, and backup networks.
* Scale cleanly in the future.

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name    | Subnet         | Purpose                              |
| ------: | ------- | -------------- | ------------------------------------ |
|      10 | SERVERS | 10.100.10.0/24 | VMs and containers only              |
|      20 | TRUSTED | 10.100.20.0/24 | Laptops, phones (Wi-Fi)              |
|      30 | GUEST   | 10.100.30.0/24 | Guest Wi-Fi                          |
|      40 | IOT     | 10.100.40.0/24 | IoT devices, printers, cameras       |
|      50 | BACKUP  | 10.100.50.0/24 | Backup NAS (TrueNAS)                 |
|      99 | MGMT    | 10.100.99.0/24 | Router, switches, APs, Proxmox hosts |

---

## VLAN Access Control Summary

VLAN 10 (Servers) hosts application workloads, including VMs and containers. It is accessible only from Trusted VLAN 20 and the management VLAN 99, and it has restricted access to Backup VLAN 50. VLAN 20 (Trusted) can access VLAN 10 (Servers) but **cannot access VLAN 50 (Backup) or VLAN 99 (Management)**. VLAN 30 (Guest) and VLAN 40 (IoT) only have Internet access and cannot reach any internal VLANs. VLAN 50 (Backup) is accessible **only** by VLAN 10 (Servers), and VLAN 99 (Management) is reserved strictly for administrative access from authorized devices.

The CAP ax AP is physically connected to the bridge and managed on VLAN 99. It bridges client traffic to VLANs 20, 30, and 40 via SSIDs. VLAN 99 is only reachable by authorized admin devices; VLAN 50 is not accessible to Wi-Fi clients.

Workload VMs, such as rsyslog, monitoring systems, or other services, reside in VLAN 10. They do not run on hypervisors or the router, and they do not require direct access to VLAN 50 or VLAN 99.

---

## IP Addressing Standards

* Gateways: `.1` of each subnet.
* DHCP pools start at `.100`.
* Static IPs for:

  * Proxmox hosts
  * Backup systems
  * Network infrastructure
  * Printers and APs

### Proxmox Host IPs (MGMT VLAN)

| Host  | IP Address   |
| ----- | ------------ |
| pve-node1 | 10.100.99.11 |
| pve-node2 | 10.100.99.12 |
| pve-node3 | 10.100.99.13 |
| pve-node4 | 10.100.99.14 |

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

**No DHCP on SERVERS, MGMT, or BACKUP VLANs.**

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

* Management IP on **VLAN 99**.
* VLAN-aware bridge (`vmbr0`) with **no IP**.
* VMs and containers tagged into VLAN 10 (SERVERS). Additional VLANs are only added if explicitly required.

This ensures:

* Hypervisors are isolated from workloads.
* Compromised VMs cannot access management VLAN.
* Firewall rules enforce strict VLAN isolation.

---

## Firewall Policy Summary

### Key Rules

* TRUSTED → SERVERS: Allowed
* TRUSTED → MGMT: Allowed for admin devices only
* TRUSTED → BACKUP: Denied
* SERVERS → BACKUP: Allowed
* SERVERS → MGMT: Denied
* GUEST / IOT → any internal VLAN: Denied

### Explicit MGMT and Backup Protection

```mikrotik
/ip firewall filter print

# Allow trusted devices to management
add chain=forward src-address=10.100.20.0/24 dst-address=10.100.99.0/24 action=accept

# Block all other access to management
add chain=forward dst-address=10.100.99.0/24 action=drop

# Allow hypervisors to backup NAS
add chain=forward src-address-list=proxmox_hosts dst-address-list=backup_nas action=accept

# Block all other access to backup
add chain=forward dst-address=10.100.50.0/24 action=drop
```

---

## Remote Management Note

Since physical access to VLAN 99 may be limited, remote administrative access can be achieved via:

* Secure VPN (e.g., Tailscale) connecting to VLAN 10 or a jump host.
* Ensuring firewall rules restrict VPN clients to only authorized devices/subnets.
* Avoiding direct exposure of VLAN 99 or VLAN 50 to Wi-Fi clients.

---

## Best Practices

* Hypervisors are **management infrastructure** and never share a subnet with workloads.
* Trunk ports for virtualization hosts.
* Default deny between VLANs.
* Static IPs for all infrastructure.
* Backup network reachable **only** from hypervisors.
* Document VLAN usage and IP assignments.
* Log aggregation and monitoring services should reside in VLAN 10.

---

## Final Notes

This design:

* Follows enterprise Proxmox deployment standards.
* Strongly limits blast radius from VM compromise.
* Keeps management isolated and auditable.
* Scales cleanly as the lab grows.
* Provides secure remote management without exposing sensitive VLANs to Wi-Fi or untrusted networks.
document.
