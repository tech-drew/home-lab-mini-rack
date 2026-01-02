# MikroTik RB5009 – Enterprise-Style VLAN Segmentation Guide


## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

### Goals
- Separate trusted users, servers, backup systems, IoT, and guest devices
- Enforce security using firewall rules (not just VLANs)
- Allow trusted Wi-Fi devices to access servers
- Isolate IoT, guest, and backup networks
- Scale easily in the future

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name     | Subnet            | Purpose |
|---------|----------|-------------------|--------|
| 10      | SERVERS  | 10.100.10.0/24    | Proxmox hosts, VMs, containers |
| 20      | TRUSTED  | 10.100.20.0/24    | Laptops, phones (Wi-Fi) |
| 30      | GUEST    | 10.100.30.0/24    | Guest Wi-Fi |
| 40      | IOT      | 10.100.40.0/24    | IoT devices, printers, cameras |
| 50      | BACKUP   | 10.100.50.0/24    | Backup NAS (TrueNAS) |
| 99      | MGMT     | 10.100.99.0/24    | Router, switch, AP management |

**Notes:**
- Gateways: `.1` of each subnet
- DHCP pools start at `.100`
- Servers and backup systems use static IPs

---

## Physical Port Layout

| Port | Device | Mode |
|------|--------|------|
| ether1 | ISP Modem | WAN |
| ether2 | CAP ax AP | Trunk (tagged VLANs) |
| ether3 | Spare | Not assigned |
| ether4 | TrueNAS Backup NAS | Access (VLAN 50) |
| ether5–ether8 | Proxmox nodes | Access (VLAN 10) |

---

## Step 1 – Create the Bridge

/interface bridge
add name=bridge vlan-filtering=yes


Add LAN ports to the bridge:

/interface bridge port
add bridge=bridge interface=ether2
add bridge=bridge interface=ether4 pvid=50
add bridge=bridge interface=ether5 pvid=10
add bridge=bridge interface=ether6 pvid=10
add bridge=bridge interface=ether7 pvid=10
add bridge=bridge interface=ether8 pvid=10


> Note: ether4 is dedicated to the backup NAS and assigned to VLAN 50.

---

## Step 2 – Define VLANs on the Bridge

/interface bridge vlan
add bridge=bridge vlan-ids=10 tagged=bridge untagged=ether5,ether6,ether7,ether8
add bridge=bridge vlan-ids=20 tagged=bridge,ether2
add bridge=bridge vlan-ids=30 tagged=bridge,ether2
add bridge=bridge vlan-ids=40 tagged=bridge,ether2
add bridge=bridge vlan-ids=50 tagged=bridge untagged=ether4
add bridge=bridge vlan-ids=99 tagged=bridge


---

## Step 3 – Create VLAN Interfaces

/interface vlan
add interface=bridge name=vlan10 vlan-id=10
add interface=bridge name=vlan20 vlan-id=20
add interface=bridge name=vlan30 vlan-id=30
add interface=bridge name=vlan40 vlan-id=40
add interface=bridge name=vlan50 vlan-id=50
add interface=bridge name=vlan99 vlan-id=99


---

## Step 4 – Assign IP Addresses

/ip address
add address=10.100.10.1/24 interface=vlan10
add address=10.100.20.1/24 interface=vlan20
add address=10.100.30.1/24 interface=vlan30
add address=10.100.40.1/24 interface=vlan40
add address=10.100.50.1/24 interface=vlan50
add address=10.100.99.1/24 interface=vlan99


---

## Step 5 – DHCP Servers

Create DHCP pools (no DHCP for SERVERS or BACKUP):

/ip pool
add name=pool20 ranges=10.100.20.100-10.100.20.200
add name=pool30 ranges=10.100.30.100-10.100.30.200
add name=pool40 ranges=10.100.40.100-10.100.40.200


Create DHCP servers:

/ip dhcp-server
add interface=vlan20 address-pool=pool20 disabled=no
add interface=vlan30 address-pool=pool30 disabled=no
add interface=vlan40 address-pool=pool40 disabled=no


---

## Step 6 – CAP ax Access Point

| SSID | VLAN |
|------|------|
| Home-WiFi | 20 |
| Guest-WiFi | 30 |
| IoT-WiFi | 40 |

Each SSID:
- VLAN mode: use-tag
- VLAN ID set accordingly

---

## Step 7 – Firewall Rules (Core Security)

### Address Lists

/ip firewall address-list
add list=trusted_wifi address=10.100.20.10 comment="Laptop"
add list=trusted_wifi address=10.100.20.11 comment="Phone"

add list=proxmox_hosts address=10.100.10.11
add list=proxmox_hosts address=10.100.10.12
add list=proxmox_hosts address=10.100.10.13
add list=proxmox_hosts address=10.100.10.14

add list=backup_nas address=10.100.50.10


---

### Firewall Filter Rules

Allow established connections:

add chain=forward connection-state=established,related action=accept


Drop invalid:

add chain=forward connection-state=invalid action=drop


Allow trusted Wi-Fi → servers:

add chain=forward src-address-list=trusted_wifi dst-address=10.100.10.0/24 action=accept


Block other Wi-Fi → servers:

add chain=forward src-address=10.100.20.0/24 dst-address=10.100.10.0/24 action=drop


Allow Proxmox → backup NAS:

add chain=forward src-address-list=proxmox_hosts dst-address-list=backup_nas action=accept


Block all other access to backup VLAN:

add chain=forward src-address=10.100.0.0/16 dst-address=10.100.50.0/24 action=drop


Block guest → internal:

add chain=forward src-address=10.100.30.0/24 dst-address=10.100.0.0/16 action=drop


Block IoT → trusted & servers:

add chain=forward src-address=10.100.40.0/24 dst-address=10.100.20.0/24 action=drop
add chain=forward src-address=10.100.40.0/24 dst-address=10.100.10.0/24 action=drop


Allow LAN → WAN:

add chain=forward out-interface=ether1 action=accept


---

## Best Practices

- Default deny between VLANs
- Static IPs for servers, backup systems, printers, and APs
- Restrict backup access to hypervisors only
- Disable device discovery when possible
- Keep RouterOS updated
- Back up router config regularly
- Document VLAN IDs, names, and IP ranges

---

## Final Notes

This design:
- Matches enterprise segmentation principles
- Protects backup infrastructure
- Is secure by default
- Scales easily
- Avoids flat-network risks

Optional future upgrades:
- VPN VLAN
- Monitoring and logging
- Hypervisor / workload VLAN split
- Backup immutability
- High availability (optional)
