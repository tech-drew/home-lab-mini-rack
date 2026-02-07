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

**Note**: Due to the global PC parts shortage in 2026, it may not be possible to acquire comparable hardware at prices similar to those listed in this document. Many components were purchased prior to the shortage or were obtained through aggressive bargain hunting on the used market afterward. As a result, the costs documented here should be treated as historical rather than representative of current market conditions.

---

## Future Expansion Plans

This section outlines planned **storage and capability expansions**. These components represent the *ideal* end state of the lab. Due to cost constraints, some or all of this functionality may initially be implemented using **repurposed hardware or improvised solutions**, with dedicated hardware added later.

Primary long-term goals:

* Reliable **off-host backups**
* **Highly available shared storage**
* Continued experimentation with virtualization, containers, and networking

---

## Future Storage Architecture

I am looking to obtain two NAS units for different purposes:

High Availability NAS: Ideally, this would be a low-power, small-sized device featuring an NVMe boot SSD and two 2TB NVMe or SATA SSDs. This setup would provide fast random read and write performance with low latency, making it ideal for high availability.

Backup NAS: For the backup unit, I would prefer a low-power, small-sized device with an NVMe boot drive and two large-capacity HDDs, such as Western Digital Red Plus NAS drives, configured in RAID 1 for data redundancy.

I am open to using any equipment I can find at a reasonable price, with a strong focus on compact size, low power consumption, and reliable performance.

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


*This project is open-source and intended for educational purposes and home lab experimentation.*
