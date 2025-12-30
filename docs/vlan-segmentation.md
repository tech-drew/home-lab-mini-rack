# MikroTik RB5009 – Enterprise-Style VLAN Segmentation Guide

## Overview

This guide describes how to configure an **enterprise-style, secure, and scalable VLAN design** on a MikroTik RB5009 using RouterOS.

### Goals
- Separate trusted users, homelab, IoT, and guest devices
- Enforce security using firewall rules (not just VLANs)
- Allow trusted Wi-Fi devices to access the homelab
- Isolate IoT and guest devices
- Scale easily in the future

---

## Network Design Summary

### VLAN Plan

| VLAN ID | Name         | Subnet            | Purpose |
|---------|-------------|------------------|--------|
| 10      | HOMELAB     | 10.100.10.0/24   | Servers, lab devices |
| 20      | TRUSTED     | 10.100.20.0/24   | Laptops, phones (Wi-Fi) |
| 30      | GUEST       | 10.100.30.0/24   | Guest Wi-Fi |
| 40      | IOT         | 10.100.40.0/24   | IoT devices, printers, cameras |
| 99      | MGMT        | 10.100.99.0/24   | Router, switch, AP management |

**Notes:**
- Gateways: `.1` of each subnet (e.g., 10.100.10.1, 10.100.20.1)
- DHCP pools start at `.100` to avoid conflicts with static IPs

---

## Physical Port Layout (Example)

| Port | Device | Mode |
|------|--------|------|
| ether1 | ISP Modem | WAN |
| ether2 | CAP ax AP | Trunk (tagged VLANs) |
| ether3 | Spare | Not assigned |
| ether4–ether8 | Homelab devices | Access (VLAN 10) |

---

## Step 1 – Create the Bridge

```

/interface bridge
add name=bridge vlan-filtering=yes

```

Add LAN ports to the bridge:

```

/interface bridge port
add bridge=bridge interface=ether2
add bridge=bridge interface=ether4 pvid=10
add bridge=bridge interface=ether5 pvid=10
add bridge=bridge interface=ether6 pvid=10
add bridge=bridge interface=ether7 pvid=10
add bridge=bridge interface=ether8 pvid=10

```

> Note: ether3 remains unassigned as a spare.

---

## Step 2 – Define VLANs on the Bridge

```

/interface bridge vlan
add bridge=bridge vlan-ids=10 tagged=bridge untagged=ether4,ether5,ether6,ether7,ether8
add bridge=bridge vlan-ids=20 tagged=bridge,ether2
add bridge=bridge vlan-ids=30 tagged=bridge,ether2
add bridge=bridge vlan-ids=40 tagged=bridge,ether2
add bridge=bridge vlan-ids=99 tagged=bridge

```

---

## Step 3 – Create VLAN Interfaces

```

/interface vlan
add interface=bridge name=vlan10 vlan-id=10
add interface=bridge name=vlan20 vlan-id=20
add interface=bridge name=vlan30 vlan-id=30
add interface=bridge name=vlan40 vlan-id=40
add interface=bridge name=vlan99 vlan-id=99

```

---

## Step 4 – Assign IP Addresses

```

/ip address
add address=10.100.10.1/24 interface=vlan10
add address=10.100.20.1/24 interface=vlan20
add address=10.100.30.1/24 interface=vlan30
add address=10.100.40.1/24 interface=vlan40
add address=10.100.99.1/24 interface=vlan99

```

---

## Step 5 – DHCP Servers

Create DHCP pools:

```

/ip pool
add name=pool10 ranges=10.100.10.100-10.100.10.200
add name=pool20 ranges=10.100.20.100-10.100.20.200
add name=pool30 ranges=10.100.30.100-10.100.30.200
add name=pool40 ranges=10.100.40.100-10.100.40.200

```

Create DHCP servers:

```

/ip dhcp-server
add interface=vlan10 address-pool=pool10 disabled=no
add interface=vlan20 address-pool=pool20 disabled=no
add interface=vlan30 address-pool=pool30 disabled=no
add interface=vlan40 address-pool=pool40 disabled=no

```

---

## Step 6 – CAP ax Access Point

Create SSIDs on the CAP ax:

| SSID | VLAN |
|------|-----|
| Home-WiFi | 20 |
| Guest-WiFi | 30 |
| IoT-WiFi | 40 |

Each SSID:
- VLAN mode: use-tag
- VLAN ID set accordingly

---

## Step 7 – Firewall Rules (Core Security)

### Address List for Trusted Devices

```

/ip firewall address-list
add list=trusted_wifi address=10.100.20.10 comment="Laptop"
add list=trusted_wifi address=10.100.20.11 comment="Phone"

```

---

### Firewall Filter Rules

Allow established connections:

```

add chain=forward connection-state=established,related action=accept

```

Drop invalid:

```

add chain=forward connection-state=invalid action=drop

```

Allow trusted Wi-Fi → homelab:

```

add chain=forward src-address-list=trusted_wifi dst-address=10.100.10.0/24 action=accept

```

Block other Wi-Fi → homelab:

```

add chain=forward src-address=10.100.20.0/24 dst-address=10.100.10.0/24 action=drop

```

Block guest → internal:

```

add chain=forward src-address=10.100.30.0/24 dst-address=10.100.0.0/16 action=drop

```

Block IoT → trusted & homelab:

```

add chain=forward src-address=10.100.40.0/24 dst-address=10.100.20.0/24 action=drop
add chain=forward src-address=10.100.40.0/24 dst-address=10.100.10.0/24 action=drop

```

Allow LAN → WAN:

```

add chain=forward out-interface=ether1 action=accept

```

---

## Printers (IoT VLAN)

- Place printers in VLAN 40
- Assign static IPs
- Allow printing from VLAN 20 only (trusted devices)

Common ports:
- TCP 9100
- TCP 631 (IPP)

---

## Best Practices

- Default deny between VLANs
- Static IPs for servers, printers, and APs
- Disable device discovery when possible
- Keep RouterOS updated
- Back up config regularly
- Document VLAN IDs, names, and IP ranges

---

## Final Notes

This design:
- Matches enterprise segmentation principles
- Is secure by default
- Scales easily
- Avoids flat-network risks

Optional future upgrades:
- VPN VLAN
- Monitoring and logging
- Additional switches
- High availability (optional)
