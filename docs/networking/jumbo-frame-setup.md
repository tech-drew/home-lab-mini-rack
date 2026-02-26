# Jumbo Frame Setup Guide

This guide explains how to configure **jumbo frames** in a home lab or enterprise-style environment. Jumbo frames (typically MTU 9000) are often used for **SANs, iSCSI storage, and high-performance networks** because they reduce CPU overhead and increase throughput.

> **Note:** All devices along the path (hosts, bridges, and routers) must have matching MTU settings for jumbo frames to work correctly.

---

## Order of Configuration

To maintain network connectivity, configure jumbo frames in the following order:

### 1. Configure PVE Host Interfaces

1. **Physical NICs (e.g., `eth0`)**

   * Set MTU to 9000.
   * Update `/etc/network/interfaces` for persistence:

```bash
auto eth0
iface eth0 inet manual
    mtu 9000
```

2. **Linux bridges (e.g., `vmbr0`)**

   * Bridges must match the NIC MTU.
   * Example configuration:

```bash
auto vmbr0
iface vmbr0 inet manual
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
    mtu 9000
```

3. **VLAN subinterfaces (optional, e.g., for iSCSI storage)**

```bash
auto vmbr0.60
iface vmbr0.60 inet static
    address 10.100.60.11/24
    mtu 9000
```

> **Tip:** Only assign IPs to VLAN subinterfaces when needed (e.g., storage traffic). Avoid placing management traffic on storage VLANs.

---

### 2. Restrict Proxmox Web GUI to Management Interface

* Prevent the GUI from being exposed on storage or other VLANs.
* Edit the `pveproxy` configuration:

```bash
# /etc/default/pveproxy
BIND_ADDRESS=10.100.99.11
```

* For newer versions: `/etc/pve/local/pveproxy.cfg` may be used instead.
* Restart the service:

```bash
systemctl restart pveproxy
```

> This ensures the GUI is only accessible via the **management VLAN**.

---

### 3. Configure MikroTik Router Ports (Connected to PVE Hosts)

1. Adjust MTU and L2MTU on the physical router interfaces:

```bash
/interface ethernet set ether1 mtu=9000 l2mtu=9216
```

2. Verify changes:

```bash
/interface ethernet print
```

> **Note:** L2MTU may need to be slightly larger than MTU to account for VLAN tagging and other overhead.

---

### 4. Adjust Bridge on MikroTik (if used)

* If the router uses a bridge connecting multiple devices, update the bridge MTU and L2MTU:

```bash
/interface bridge set bridge1 mtu=9000 l2mtu=9216
```

* Verify bridge settings:

```bash
/interface bridge print
```

---

### 5. Testing Jumbo Frames

* Use ping to test connectivity with jumbo frames:

```bash
ping 10.100.60.15 size=8972 df-bit
```

* **Calculation:** MTU – 28 bytes (ICMP + IP header) = payload size.

* Confirm there are **no packet drops**.

---

### 6. Best Practices

* **Consistency:** All devices on the same network path must match MTU settings.
* **Isolation:** Keep storage traffic separate from management traffic (dedicated VLAN or NIC).
* **Firewall:** Allow iSCSI ports (TCP 3260) on storage VLAN; block GUI/SSH on storage VLAN.
* **Incremental changes:** Update devices in the order above to avoid connectivity loss.

---

This setup ensures **high-performance storage networking** while maintaining **secure management access**.
