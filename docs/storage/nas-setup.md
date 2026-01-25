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

## Backup NAS — 

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
