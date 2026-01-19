# Storage Setup Documentation

## Overview

This infrastructure uses **two NAS enclosures**, each serving a distinct purpose:

1. **Primary Backup NAS**

   * Dedicated to backing up Proxmox VMs, containers, and related infrastructure.
   * Runs **TrueNAS on bare metal**, prioritizing **data integrity**, **reliability**, and incremental expandability.

2. **High-Availability NAS**

   * Provides **high-performance, highly available storage** for Proxmox VMs, containers, and supporting infrastructure.
   * Runs **Proxmox on bare metal**, optimized for **low latency and high IOPS workloads**.

---

## Backup NAS — Aoostar WTR Pro (TrueNAS)

> **Important Note:** On the AMD Ryzen WTR Pro, the second SATA controller is integrated into the CPU/SoC. Passing the **entire second SATA controller** to a VM (e.g., TrueNAS) causes the host to lose CPU sensor access, throttling the CPU (~1.9 GHz). The **first SATA controller** can be passed through safely. This is a hardware limitation with no known fix.
> **Recommendation:** Avoid full controller passthrough. Bare-metal use or individual drive passthrough is safe.

### Hardware

* **NVMe Slots:** 2
* **3.5" HDD Bays:** 4
* **Memory:** 32 GB DDR4 (ECC not used; acceptable for home lab)

### Current Disk Usage

**NVMe Slots**

* Slot 1: TrueNAS OS boot device
* Slot 2: Reserved for future expansion or cache

**HDD Bays**

* Bay 1 & 2: 2 × 3.5" HDDs in **RAID1 mirror** for backup storage
* Bay 3 & 4: Empty (planned for future expansion)

---

### Storage Configuration

* **Boot Device:** Dedicated NVMe SSD for TrueNAS OS (separates OS from data for easier recovery/upgrades).
* **Backup Pool:** RAID1 mirror with current 2 × 3.5" HDDs, prioritizing **redundancy and data integrity** over performance.
* **Planned Expansion:** Additional drives will be added as needed; storage may migrate to **RAIDZ1 (single parity)** or **RAIDZ2 (dual parity)** depending on disk count and fault-tolerance requirements.

---

### Software Stack

* **Operating System:** TrueNAS (bare metal)
* **Backup Sources:** Proxmox VMs, containers, and infrastructure data
* Backups managed directly from Proxmox, stored on **ZFS datasets**

---

## High-Availability NAS — Aoostar WTR Pro (Proxmox)

> **Note:** Same SATA passthrough limitation applies. Avoid passing through the second SATA controller to a VM.

### Hardware

* **NVMe Slots:** 2
* **3.5" HDD Bays:** 4
* **Memory:** 32 GB DDR4 (ECC not used)

### Current Disk Usage

**NVMe Slots**

* Slots 1 & 2: 2 × 2 TB Samsung 980 Pro SSDs in a single RAID1 mirror pool for HA storage

**HDD Bays**

* Bay 1: SATA SSD for Proxmox boot
* Bays 2–4: Empty (planned for expansion)

---
### Storage Configuration

- **Boot Device:** Dedicated SATA SSD for Proxmox OS  
- **HA Storage Pool:** RAID1 mirror of NVMe SSDs, optimized for **low-latency, high random read/write workloads**  
> **Note:** NVMe SSDs are recommended for high-availability storage because HA workloads rely heavily on low-latency random read and write performance. Using HDDs for this pool could significantly reduce VM responsiveness and overall HA effectiveness.

---

### Software Stack

* **Operating System:** Proxmox (bare metal)
* **HA Storage Usage:** Proxmox VMs, containers, and supporting infrastructure
* Stored on **ZFS datasets** for redundancy and performance

---

## Additional / Future Use Cases

* **Self-hosted notes/knowledge management** (Evernote-like)
* **File storage/sync services** (self-hosted Google Drive alternative)

> **Network Considerations:** NAS units currently reside on a **dedicated backups VLAN**. Additional services may require VLAN/firewall adjustments to maintain isolation.

---

## Design Decisions

* **TrueNAS on Bare Metal:** Ensures direct hardware access, ZFS integrity, and simpler setup
* **RAID1 Start:** Minimal disks required, simple redundancy
* **Gradual Expansion:** Storage added only as needed, controlling cost
* **ECC Memory:** Not used due to budget/home lab context; enterprise environments would enable ECC

---

## Notes

* Primary focus is **data protection and backups**
* Additional services must **not compromise isolation or reliability**
* Restore integrity is **prioritized over raw performance**
