# Home Lab: Mini-rack

This project documents the creation of a compact home lab setup, designed for **self-hosting resources** and **studying technology**. The goal is to build a fully functional mini-rack that supports networking, compute nodes, and power management in a compact and energy-efficient form factor.

---

## Project Status: Work in Progress

This repository documents the planning and design of my personal home lab project.
It is currently a work in progress and includes design notes, hardware plans, and intended setup steps.

Updates will be added as the lab is built, configured, and iterated on.

---

## Hardware Components

**Notes:** This homelab is being assembled for approximately **$230** by scavenging parts and jury-rigging whatever hardware I already own or can acquire cheaply.

### 1. Modem

* **Model:** Arris Surfboard S33 (already owned)

---

### 2. Router

* **Model:** MikroTik RB5009 (already owned)
* **Price:** ~$260 (typical retail)
* **Note:** Extensive research went into selecting the networking equipment for this project. The reasoning and analysis behind choosing the MikroTik RB5009 and cAP ax can be found in the [Networking Equipment Analysis](docs/networking/networking-equipment-analysis.md).

---

### 3. Access Points

* **Model:** MikroTik cAP ax (already owned)
* **Price:** ~$120 each (typical retail)
* **Link:** [MikroTik cAP ax](https://mikrotik.com/product/cap_ax)
* **Note:** See the [Networking Equipment Analysis](docs/networking/networking-equipment-analysis.md) for design reasoning.

---

### 4. Compute Nodes

* **Model:** Dell Wyse 5070 Thin Client × 4
* **Price:** ~$155 total (used / purchased 11/28/2025 on eBay)
* **Link:** [eBay Wyse 5070](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client&_sacat=0)

**Upgrades**

* 32 GB DDR4 RAM (reused from previous projects)
* 256 GB M.2 SATA SSDs (see storage section)

**Storage Compatibility**

* NVMe SSDs are **not supported**
* Requires M.2 SATA 2260 or 2280 SSDs

**Power Draw**

* Idle: ~5–6 W
* Light load: 7–10 W
* Full CPU load: 13–15 W

**Notes**

* Updating to the latest BIOS is recommended for stability.
* These systems are reported to be reliable for 24/7 operation.
* The Intel J5005 memory controller can efficiently address ~30 GB of RAM.

**Memory Behavior**

* **Linux:** Installing 32 GB results in ~30 GB usable. The remaining memory is unaddressed but does not affect stability.
* **Windows:** To use 32 GB:

  * Secure Boot must be disabled
  * The following boot option must be added:

    ```
    bcdedit /set {current} truncatememory 0x800000000
    ```
  * Windows will then report ~29.8 GB usable
* The limitation is due to the CPU memory controller; memory rank configuration does not change this behavior.

---

### 5. Storage for Wyse Clients

* **Model:** SK Hynix 256 GB M.2 2280 SATA SSD
* **Price:** ~$74 total (used / purchased 12/13/2025 on eBay)
* **Link:** [SK Hynix 256 GB M.2 SATA SSD](https://www.ebay.com/itm/388921986094)

**Notes**

* Fully compatible with the Wyse 5070 SATA-only M.2 slot
* Used SSDs were considered but rejected due to wear concerns at minimal cost savings
* Adequate for Linux, Proxmox, Docker, and lightweight Kubernetes workloads
* SSD brand is not critical; selection was based on value and reliability
* SMART monitoring will be used to track drive health

---

### 6. UPS

* **Model:** APC BN1500M2 (already owned)
* **Notes:** A pure sine wave UPS is a future upgrade. This unit is being used temporarily to keep costs down.

---

### Total Cost of Purchased Materials

| Item                      | Cost (USD) |
| ------------------------- | ---------- |
| Compute Nodes             | $155       |
| Storage (4 × 256 GB SSDs) | $74        |
| **Total**                 | **$229**   |

---

## Future Expansion Plans

This section outlines planned **storage and capability expansions**. These components represent the *ideal* end state of the lab. Due to cost constraints, some or all of this functionality may initially be implemented using **repurposed hardware or improvised solutions**, with dedicated hardware added later.

Primary long-term goals:

* Reliable **off-host backups**
* **Highly available shared storage**
* Continued experimentation with virtualization, containers, and networking

---

## Future Storage Architecture

### Dedicated NAS for Backups

* **Target Model:** Aoostar WTR Pro
* **Estimated Cost:** ~$400
* **Link:** [Aoostar WTR Pro](https://aoostar.com/collections/nas-series/products/aoostar-wtr-pro-4-bay-90t-storage-amd-ryzen-7-5825u-nas-mini-pc-support-2-5-3-5-hdd-%E5%A4%8D%E5%88%B6)

**Purpose**

* Centralized backup destination for VMs, containers, and configuration data

**Planned Configuration**

* Start with **RAID1** using two drives
* Leave remaining bays empty for expansion
* Migrate to **RAID-Z1 or RAID-Z2** as storage needs increase

This system will be isolated from production workloads to reduce risk and simplify backup and recovery.

---

### NAS for Shared Storage & High Availability

* **Target Model:** Aoostar WTR Pro
* **Estimated Cost:** ~$400

**Purpose**

* Shared storage for Proxmox to enable live migration, HA, and clustered workloads

**Design Rationale**

* Separating backup and production storage reduces failure impact
* Dual NAS design simplifies VLAN segmentation
* Mirrors real-world infrastructure practices

**Planned Configuration**

* Initial **RAID1**
* Expandable to **RAID-Z1 or RAID-Z2**

---

### NAS Hard Drives

* **Target Model:** Western Digital Red Plus

**Notes**

* Designed for NAS workloads and 24/7 operation
* Drive size and quantity will be selected after real usage data is available
* Reclaimed or mixed-capacity drives may be used initially to control costs

---

## Future Software & Lab Capabilities

As the lab matures, focus will shift from hardware assembly to **platform experimentation**:

### Virtualization & Containers

* Proxmox VE
* Kubernetes (lightweight or full)
* Docker / Docker Swarm

### Self-Hosted Services

* Backup automation and snapshotting
* Monitoring and logging stacks
* Services such as Nextcloud, Home Assistant, and CI/CD tools

### Networking Experiments

* VLAN segmentation using MikroTik RB5009
* Firewall rules, VPNs, and routing labs
* Simulated multi-tier network environments

---

## Trade-offs and Limitations

This lab prioritizes **learning, efficiency, and cost control** over raw performance.

* Thin clients offer limited CPU power but extremely low energy usage
* Storage capacity is intentionally conservative
* Managed PDUs and enterprise-grade hardware are deferred due to cost
* Incremental upgrades are preferred over large upfront investments

---

## Design Philosophy

This project favors **learning through constraints**. Temporary or improvised solutions are intentional and part of the experimentation process. Hardware and architecture decisions will evolve based on real-world experience rather than assumptions.

---

## References / Links

* [Dell Wyse 5070 Thin Client](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client)

---

*This project is open-source and intended for educational purposes and home lab experimentation.*
