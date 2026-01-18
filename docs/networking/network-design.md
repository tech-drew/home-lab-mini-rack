# Network Design Summary: Secure & Scalable MikroTik RB5009

This guide describes the configuration of a secure, high-performance VLAN architecture on a MikroTik RB5009. It follows **CIS Critical Security Controls** by disabling VLAN 1 and utilizing a **Native Sink (999)** to neutralize untagged traffic on exposed trunk ports.

---

## Goals

* **Create an enterprise-style secure and scalable VLAN Design**
* **Follow Enterprise Networking and CyberSecurity Best Practices**
* **Maximize storage backplane performance with Jumbo Frames**
* **Implement "Early Warning" physical intrusion detection**
* **Balance Security (Native Sink) with Availability (Management PVIDs)**

---

## VLANs

The table defines the core network services and addressing for each VLAN. By restricting DHCP and DNS on infrastructure-critical VLANs (50, 60, 99) we reduce attack surface on critical VLANs.

| VLAN ID | Name | Subnet | DHCP | DNS | Purpose / Service Logic |
| --- | --- | --- | --- | --- | --- |
| **10** | SERVERS | 10.100.10.0/24 | **Yes** | **Yes** | VMs and containers only. |
| **20** | TRUSTED | 10.100.20.0/24 | **Yes** | **Yes** | Trusted devices (Admin laptops, Phones). |
| **30** | GUEST | 10.100.30.0/24 | **Yes** | **Yes** | Isolated visitors; WAN access only. |
| **40** | IOT | 10.100.40.0/24 | **Yes** | **Yes** | IoT devices, printers, cameras. |
| **50** | BACKUP | 10.100.50.0/24 | **No** | **No** | **Static Only.** Backup NAS (TrueNAS). |
| **60** | STORAGE | 10.100.60.0/24 | **No** | **No** | **Static Only.** HA Storage / Proxmox Backplane. |
| **99** | MGMT | 10.100.99.0/24 | **No*** | **No** | **Static Only.** Router, Switches, APs, PVE Hosts. |
| **999** | SINK | None | **No** | **No** | Dead-end for Untagged Trunk Traffic. |

> **Note on Management (VLAN 99):** While documented as **Static Only** for production reliability, a small temporary DHCP pool can be enabled during "staging" to provision new hardware before assigning static leases.

---

## Architectural Rationale: Mixed MTU & Traffic Flow

This design utilizes a **Mixed MTU environment** to balance high-performance infrastructure with universal client compatibility. The strategy is built on the distinction between how data moves within the cluster versus how it moves to the user.

### 1. East-West Traffic (Inter-Host Storage)

* **Direction:** Sideways movement between Proxmox hosts (e.g., Host A to Host B).
* **MTU:** Native **9000 (Jumbo Frames)**.
* **Purpose:** High-volume operations including **Storage Replication, ZFS Send/Receive, and Live VM Migrations**.
* **The Efficiency Advantage:** Moving 9,000 bytes of data in a single frame significantly reduces the ratio of overhead to actual data.

#### **Efficiency Comparison: Moving 9,000 Bytes of Data**

| Metric | Standard MTU 1500 | Jumbo MTU 9000 | Difference |
| --- | --- | --- | --- |
| **Frames Required** | 6 | 1 | **83.3% Fewer Frames** |
| **Total Header Overhead** | 228 Bytes | 38 Bytes | **190 Byte Savings** |
| **CPU Interrupts** | 6 | 1 | **83.3% Lower Load** |
| **Effective Payload** | 1,500 bytes/frame | 9,000 bytes/frame | **6x Data Density** |

### 2. North-South Traffic (Client-to-Server)

* **Direction:** Vertical movement from a user/client to a service (e.g., Laptop to VM).
* **MTU:** Standard **1500**.
* **Purpose:** General access, internet browsing, and management.
* **The "Fragmentation Tax":** When a device in a 1500 MTU VLAN requests data from the 9000 MTU Storage VLAN, the RB5009 performs Layer 3 fragmentation.

#### **Fragmentation Tax Analysis (Per 9,000 Byte Transaction)**

| Operation | Logic | Result |
| --- | --- | --- |
| **Ingress** | Router receives 1x 9000-byte frame from Storage. | **1 Interrupt** |
| **Translation** | Router CPU calculates 6x 1500-byte segments + new headers. | **CPU Overhead** |
| **Egress** | Router transmits 6x 1500-byte frames to Client. | **6 Interrupts** |
| **Total Impact** | Router processes **7 total interrupts** for one transaction. | **16.6% Higher Load** |

### 3. Jumbo Frames: Pros and Cons

While mathematically superior for throughput, Jumbo Frames require strict adherence to configuration standards to avoid network instability.

| Pros | Cons |
| --- | --- |
| **Increased Throughput:** Higher data-to-header ratio maximizes bandwidth. | **End-to-End Requirement:** Every switch and NIC in the path must support MTU 9000 or packets will be dropped. |
| **Lower CPU Overhead:** Fewer interrupts allow the host CPU to focus on VM workloads. | **Difficult Troubleshooting:** MTU mismatches often cause "zombie connections" (ping works, but large data transfers fail). |
| **Reduced Latency:** Fewer packets mean less time spent in switch buffers and queues. | **Standardization Issues:** Some consumer gear or cheap NICs do not support frames larger than 1500 or 4000 bytes. |

### 4. Security vs. Visibility (The Native Sink)

This design prioritizes **Prevention** via the **NATIVE_SINK (VLAN 999)** rather than a "Honeypot" detection approach.

* **The "Wall" Approach:** By sinking untagged traffic, an unauthorized device gains zero network footprint—no IP, no gateway, and no internet access.
* **Visibility via Logging:** Unauthorized access attempts are captured at the hardware level via **RouterOS Logging**. This provides "Early Warning" detection of physical intrusions without expanding the network's attack surface.

---

### **Summary of Benefits**

* **Maximum Performance:** Storage replication (East-West) runs at near-theoretical wire speed.
* **Maximum Compatibility:** Users (North-South) access services via standard 1500 MTU with no special configuration.
* **Maximum Security:** Unauthorized ports are non-functional "dead ends" by default.

---

## Wireless Access Design (CAPsMAN)

Wireless networking in this environment is implemented as a **Layer 2 access extension** of the existing VLAN architecture. All wireless policy enforcement, segmentation, and security controls are inherited directly from the wired network design. No wireless-specific trust zones or routing exceptions are introduced.

Wireless access points are centrally managed by **CAPsMAN** running on the RB5009. The APs themselves do not perform routing, NAT, or firewall functions.

---

### Wireless Design Principles

* **Single Source of Truth:** VLAN definitions, trust boundaries, and firewall policy are defined once and enforced consistently across wired and wireless access.
* **No Wireless-Only VLANs:** All SSIDs map directly to pre-existing VLANs.
* **Centralized Control Plane:** CAPsMAN provides unified SSID configuration, security profiles, and radio management.
* **Infrastructure Isolation:** AP management traffic is isolated from client traffic using VLAN separation.
* **Zero Trust by Default:** Wireless clients are treated identically to wired clients within the same VLAN.

**Note:** MikroTik wireless requires additional setup and tuning to achieve smooth roaming and consistent behavior across multiple access points. For now, the wireless network is implemented to provide reliable connectivity, proper network separation, and centralized management. More advanced wireless optimizations will be added and documented later as the environment matures.

---

### SSID to VLAN Mapping

| SSID Name    | VLAN ID | VLAN Name | Purpose                                |
| ------------ | ------: | --------- | -------------------------------------- |
| HOME-TRUSTED |      20 | TRUSTED   | Primary user devices (laptops, phones) |
| HOME-GUEST   |      30 | GUEST     | Guest access (WAN only)                |
| HOME-IOT     |      40 | IOT       | IoT devices and embedded systems       |

All SSIDs apply **802.1Q VLAN tagging at the AP**, ensuring traffic is properly segmented before entering the wired network.

---

### Access Point Management & Control Plane

* AP management interfaces reside exclusively in **VLAN 99 (MGMT)**.
* CAPsMAN control traffic operates entirely within the MGMT VLAN.
* APs are not reachable from Guest (30) or IoT (40) VLANs.
* Only Trusted (20) and MGMT (99) devices may administer wireless infrastructure.

This ensures that compromise of a wireless client does not provide a path to infrastructure management.

---

### Wired Uplink and Trunk Configuration

Access points connect to the RB5009 using **802.1Q trunk ports** with strict ingress filtering:

* **Tagged VLANs:** 10, 20, 30, 40, 99  
  *(VLANs 50 and 60 are intentionally omitted, as they are not accessible via wireless.)*
* **Native VLAN (PVID):** 999 (NATIVE_SINK)
* **Untagged traffic:** Dropped via the Native Sink VLAN
* **Ingress Filtering:** Enabled (only VLANs tagged on this port are allowed; all others are dropped)

This design prevents:

* VLAN hopping
* Double-tagging attacks
* Accidental untagged client bridging

---

### Security Considerations

* Wireless clients inherit **identical firewall rules** as wired clients within the same VLAN.
* Guest and IoT SSIDs are fully isolated and restricted to WAN access only.
* No Layer 3 interface exists on the **NATIVE_SINK (999)** VLAN.
* CAPsMAN configuration changes follow the same Safe Mode and rollback procedures as core router changes.

---

### Operational Scope

The network design document intentionally excludes low-level RF and radio configuration details such as:

* Channel selection
* Transmit power tuning
* Band steering
* Roaming optimization
* Regulatory domain settings

These parameters are considered **operational implementation details** and are maintained within CAPsMAN configuration and RouterOS comments.

---

### Summary

Wireless access in this homelab is a **first-class citizen of the network architecture**, not a parallel or exception-based system. By enforcing consistent VLAN tagging, centralized management, and strict isolation of infrastructure traffic, the wireless design aligns fully with enterprise networking and security best practices.

## Physical Port Layout (RB5009)

Ports are categorized by **Risk Profile**. Exposed ports (APs) are "Sunk," while internal infrastructure ports (NAS/Proxmox) use Management or Data PVIDs for fail-safe access.

| Port | Device | Mode | Native VLAN (PVID) |
| --- | --- | --- | --- |
| ether1 | ISP Modem | WAN | N/A |
| ether2 | CAP ax AP | Security Trunk | 999 (NATIVE_SINK) |
| **ether3** | Backup NAS | Access (VLAN 50) | 50 (BACKUP) |
| **ether4** | HA Storage | Access (VLAN 60) | 60 (STORAGE) |
| ether5–ether8 | Proxmox Cluster | Infrastructure Trunk | 99 (MGMT) |

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

### 1. Global Bridge & MTU

```routeros
/interface bridge
set [find name=bridge] protocol-mode=none mtu=9000 l2mtu=9000 vlan-filtering=yes

```

### 2. Early Warning Detection (Native Sink)

```routeros
/interface vlan
add interface=bridge name=VLAN999_SINK vlan-id=999

/ip firewall filter
add action=log chain=input in-interface=VLAN999_SINK log=yes log-prefix="INTRUSION_ALERT" \
    comment="Early Warning: Device detected on Sunk Port"

```

### 3. Port Assignment

```routeros
# Security Trunks (Sunk)
set [find interface=ether2] pvid=999 frame-types=admit-only-vlan-tagged ingress-filtering=yes

# Dedicated Storage Access Ports (Untagged for the NAS)
set [find interface=ether3] pvid=50 ingress-filtering=yes
set [find interface=ether4] pvid=60 ingress-filtering=yes

# Infrastructure Trunks (Management Native)
set [find interface=ether5,ether6,ether7,ether8] pvid=99 ingress-filtering=yes

```

---

## Best Practices

1. **Risk-Based PVIDs:** Exposed ports use **VLAN 999** (Sunk). Internal storage/server ports use their native VLANs to ensure a recovery path if software tagging fails.
2. **Storage Reliability:** ether3 and ether4 are set as access ports. This ensures your NAS units are reachable even if you haven't configured VLAN tagging on the TrueNAS/Storage OS side yet.
3. **Fragmentation Awareness:** The RB5009 handles the 9000-to-1500 translation for users; the storage backplane remains at peak efficiency.
