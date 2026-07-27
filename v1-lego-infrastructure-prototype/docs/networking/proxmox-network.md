# Proxmox Network Configuration

This document explains the VLAN-aware network setup for Proxmox nodes 1–4 in the homelab, including the reasoning behind bridge and VLAN choices.

---

## Physical NICs

* **eth0** – Physical network interface used for all VLAN traffic.

**Note:** During Proxmox setup, I consistently named the network adapter used for network connectivity **eth0** on all my nodes. It is recommended to use consistent naming for your network interfaces during Proxmox installation to simplify management and documentation.

* Chosen because the router (RB5009) ports are configured to **admit only tagged traffic**.
* No untagged traffic is allowed, so the host cannot receive an IP directly on `eth0`.

> This setup is applied consistently across all four nodes to ensure predictable VLAN behavior and simplify administration.

---

## Bridge and VLAN Configuration

Proxmox uses **vmbr0**, a VLAN-aware bridge, to handle all tagged traffic. The bridge allows virtual machines and containers to access multiple VLANs over a single physical interface (`eth0`), consistent with the router’s tagged-only port policy.

### `/etc/network/interfaces` snippet

```bash
# Loopback
auto lo
iface lo inet loopback

# Physical interface
iface eth0 inet manual  # No IP on eth0 because all traffic is VLAN-tagged

# VLAN-aware bridge
auto vmbr0
iface vmbr0 inet manual
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094  # Allow all VLANs except default VLAN 1

# Management VLAN 99
auto vmbr0.99
iface vmbr0.99 inet static
    address 10.100.99.11/24  # Chosen management IP for this node
    gateway 10.100.99.1

# Wireless interface (unused)
iface wlan0 inet manual

# Include additional interface configs
source /etc/network/interfaces.d/*
```

> **Note:** To adjust this configuration for your environment, replace `10.100.99.11` with the management IP you want for your device on a given VLAN, and replace `eth0` with the name of your physical NIC.

---

### **Explanation**

1. **`eth0` in manual mode**

   * Prevents Proxmox from assigning an IP to the physical NIC.
   * Only tagged traffic flows through this interface because the router blocks untagged frames.

2. **`vmbr0` as a VLAN-aware bridge**

   * Handles tagged traffic for all VLANs (e.g., 20, 30, 40, 99).
   * `bridge-vlan-aware yes` ensures VLAN tags are preserved and recognized.
   * `bridge-vids 2-4094` allows all VLANs except the default VLAN 1, which is unused.

3. **`vmbr0.99` for management**

   * Dedicated interface for VLAN 99, used for Proxmox node management.
   * Provides reliable access even if other VLANs or VMs are misconfigured.
   * The IP `10.100.99.11` is the management IP assigned to this node.

4. **Why this is necessary**

   * Router ports only allow **tagged traffic**, so the Proxmox host must also send and receive VLAN-tagged frames to communicate.
   * Using a VLAN-aware bridge ensures all VMs and containers can access the correct VLANs without requiring additional physical NICs.

---

### **Benefits**

* **Consistent VLAN configuration across nodes** – Simplifies VM migration and troubleshooting.
* **VLAN isolation** – Keeps management, storage, and VM networks separate.
* **Fail-safe management network** – VLAN 99 provides guaranteed access to node administration.
* **Scalable** – Adding more VLANs does not require additional physical connections.

---

### **Example Usage**

* VM on VLAN 20 (Trusted) → attach NIC to `vmbr0` and tag VLAN 20.
* VM on VLAN 30 (Guest) → attach NIC to `vmbr0` with VLAN 30 tag.
* Host management → use `vmbr0.99` for SSH/Web access to the Proxmox UI.
