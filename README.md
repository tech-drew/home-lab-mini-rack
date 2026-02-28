# Home Lab: Mini-Rack

The primary objective of this home lab is to design and implement an environment that closely replicates enterprise IT infrastructure in both architecture and operational practices.

In addition to this core objective, the project emphasizes several secondary priorities: minimizing physical footprint, optimizing energy efficiency, reducing noise levels, and maintaining strict cost control.


---

## Project Status: Work in Progress

This repository documents the planning and design of my personal home lab project. It is currently a work in progress and includes design notes, hardware plans, and intended setup steps.

Updates will be added as the lab is built, configured, and iterated on. Some documentation may become out of date over time. Once everything is setup correctly and operational I will clean up documentation issues.

---

# Hardware Components

**Note:** This home lab is being assembled for approximately **$1,104 total out-of-pocket** by scavenging parts and repurposing hardware I already own or can acquire inexpensively.

---

## 1. Modem

* **Model:** Arris Surfboard S33
* **Cost:** Already owned

---

## 2. Router

* **Model:** MikroTik RB5009
* **Cost:** Already owned (~$260 typical retail)
* **Note:** Extensive research went into selecting the networking equipment. The reasoning behind choosing the RB5009 and cAP ax is documented in the *Networking Equipment Analysis*.

---

## 3. Access Points

* **Model:** MikroTik cAP ax
* **Cost:** Already owned (~$120 each typical retail)
* **Note:** See the *Networking Equipment Analysis* for design reasoning.

---

## 4. Proxmox Virtual Environment Compute Nodes

* **Model:** Dell Wyse 5070 Thin Client (×4)
* **Cost:** $155 total (used, purchased 11/28/2025 via eBay)

### Upgrades

* 32 GB DDR4 RAM (reused)
* 256 GB M.2 SATA SSDs (see Storage section)

### Storage Compatibility

* NVMe SSDs are **not supported**
* Requires M.2 SATA (2260 or 2280)

### Power Draw (per node)

* Idle: ~5–6 W
* Light load: 7–10 W
* Full CPU load: 13–15 W

### Notes

* Updating to the latest BIOS is recommended.
* These systems are considered reliable for 24/7 operation.
* The Intel J5005 memory controller can address ~30 GB of RAM.

### Memory Behavior

**Linux:** Installing 32 GB results in ~30 GB usable. The remaining memory is unaddressable but does not affect stability.

**Windows:** To utilize 32 GB:

* Disable Secure Boot
* Add the following boot option:

```
bcdedit /set {current} truncatememory 0x800000000
```

Windows will then report ~29.8 GB usable.

The limitation is due to the CPU memory controller and is unaffected by memory rank configuration.

---

## 5. Storage for Wyse Clients

* **Model:** SK Hynix 256 GB M.2 2280 SATA SSD (×4)
* **Cost:** $74 total (used, purchased 12/13/2025 via eBay)

### Notes

* Fully compatible with the Wyse 5070 SATA-only M.2 slot
* Used SSDs were rejected due to wear concerns
* Adequate for Proxmox, Docker, and lightweight Kubernetes workloads
* SMART monitoring will be used to track drive health

---

## 6. Proxmox Storage Node & Backup Server

* **Model:** HP EliteDesk G4 SFF (×2)
* **Cost:** $300 total (used, purchased 02/06/2026 via eBay)

### CPU Configuration

| System Role        | CPU                    | Cost |
| ------------------ | ---------------------- | ---- |
| Storage / NAS Node | i7-8700T (6C/12T, 35W) | $80  |
| Backup Server      | i5-8500T (6C/6T, 35W)  | $45  |

**Total CPU upgrade cost:** $125

The systems originally shipped with i5-8500 (65W) CPUs, which were replaced with 35W T-series CPUs to reduce power consumption.

### Storage Configuration

* Boot drives: 256 GB SATA SSD (×2 total) — $50 total
* Data drives: 2 TB SATA SSD (×4 total) — $400 total

  * Two drives per system in a mirrored configuration for redundancy

### Power Draw (per system)

* Idle: ~18–25 W
* Light load: 30–45 W
* Full CPU load: 65–90 W

*(Power draw is significantly higher than the Wyse thin clients due to desktop-class chipset, PSU overhead, and additional storage devices.)*

### Notes

* BIOS update recommended.
* Reliable for 24/7 operation.
* Each system includes:

  * Two 3.5" drive bays
  * Two NVMe slots on the motherboard

---

## 7. UPS

* **Model:** APC BN1500M2
* **Cost:** Already owned
* **Note:** A pure sine wave UPS is a future upgrade.

---

# Total Cost of Purchased Materials

| Item                               | Cost (USD) |
| ---------------------------------- | ---------- |
| Dell Wyse 5070 Thin Clients (×4)   | $155       |
| 256 GB M.2 SATA SSDs (×4)          | $74        |
| HP EliteDesk G4 SFF Systems (×2)   | $300       |
| CPU Upgrades (i7-8700T + i5-8500T) | $125       |
| 256 GB SATA SSDs (×2)              | $50        |
| 2 TB SATA SSDs (×4)                | $400       |
| **Total**                          | **$1,104** |

---

### Pricing Disclaimer

Due to the global PC parts shortage in 2026, comparable hardware may not be available at similar prices. Many components were purchased prior to the shortage or obtained through aggressive used-market sourcing. These figures should be considered historical and not reflective of current market conditions.

---

# Virtualization & Containers

* Proxmox VE
* Kubernetes (lightweight or full)
* Docker / Docker Swarm

---

# Self-Hosted Services

* Backup automation and snapshotting
* Monitoring and logging stacks
* Services such as Nextcloud, Home Assistant, and CI/CD tools

---

# Networking Experiments

* VLAN segmentation using MikroTik RB5009
* Firewall rules, VPNs, and routing labs
* Simulated multi-tier network environments

---

# Trade-offs and Limitations

This lab prioritizes **learning, efficiency, and cost control** over raw performance.

* Thin clients provide limited CPU power but extremely low energy usage
* Storage capacity is intentionally conservative
* Enterprise-grade hardware is deferred due to cost
* Incremental upgrades are preferred over large upfront investments

---

# Design Philosophy

This project embraces **learning through experimentation and iteration**. There are always unknowns in system design, and the most effective way to address them is through hands-on experience. The lab will evolve through cycles of building, testing, failure, refinement, and incremental improvement.

Significant financial investment is intentionally deferred until practical experimentation clarifies real-world requirements. As a result, Version 1 of this home lab relies on **juryrigged and scavenged** hardware sourced from whatever is readily available. Juryrigging and scavenging components is part of the learning process and adds to the enjoyment of building the lab.

**Note:** Given current budget constraints, the present implementation is considered successful. However, a future revision would incorporate two major changes.

First, the networking infrastructure would be upgraded. A 1 GbE network represents a hard limitation, particularly for storage-heavy or distributed workloads. At a minimum, 2.5 GbE would be deployed throughout the environment as a budget-conscious improvement. Ideally, 10 GbE would serve as the baseline, with 25 GbE or higher considered if implementing distributed storage solutions such as Ceph, where replication traffic and cluster communication significantly increase bandwidth demands.

Second, greater emphasis would be placed on hardware platforms that provide PCIe expansion for long-term flexibility. Compact systems with a PCIe slot would allow for higher-speed networking, additional storage controllers, or other specialized hardware. However, these improvements would come at the cost of increased power consumption and heat generation, requiring careful evaluation of efficiency, thermals, and overall operating expense.
