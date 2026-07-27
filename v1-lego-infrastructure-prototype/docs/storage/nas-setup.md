# Storage Setup Documentation

## Overview

This infrastructure uses **two NAS systems**, each with a clearly defined role and failure domain. Storage responsibilities are intentionally separated to reduce blast radius, simplify recovery, and allow each system to be optimized for its workload.

1. **Primary Backup NAS**

   * Dedicated to **backups and long-term data protection**
   * Optimized for **integrity, redundancy, and recoverability**
   * Runs **TrueNAS on bare metal**

2. **High-Availability NAS**

   * Provides **shared storage** for Proxmox VMs and containers
   * Optimized for **low latency and predictable I/O**
   * Runs **Proxmox on bare metal**
   * Exposes storage to the cluster over the network

All VM and container disks consumed by the Proxmox cluster reside on **ZFS-backed storage hosted by the NAS systems**, not on the compute nodes.

---

## ZFS Design Rationale (1 GbE–Limited Environment)

This home lab has a **hard 1 GbE network limit**, making bandwidth—not disk speed—the primary bottleneck. ZFS is used on the NAS systems to **minimize unnecessary network traffic**, provide predictable storage behavior, and reduce backup and replication overhead.

---

### Compression Reduces Network Traffic

ZFS applies fast, transparent compression (typically `lz4`) before data is written to disk and during replication or backup operations.

* VM and container data often compresses **1.5×–2×**
* Less data traverses the network
* CPU overhead is shifted to NAS nodes, which have sufficient capacity

---

### Snapshot-Based Backups and Incremental Replication

ZFS snapshots are lightweight and enable **efficient, incremental backups**. Only changed blocks are transferred over the network, significantly reducing traffic.

**Example – 20 GB VM Disk:**

| Feature                         | ZFS                                            | LVM-thin / ext4 / NTFS                                                         |
| ------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------ |
| Total VM Disk                   | 20 GB                                          | 20 GB                                                                          |
| Data Changed Since Last Backup  | 2 GB                                           | 2 GB                                                                           |
| Data Sent on Backup/Replication | ~2 GB (only changed blocks)                    | 20 GB (entire volume often copied)                                             |
| Network Impact                  | Minimal, fits 1 GbE easily                     | High, saturates network and slows other operations                             |
| Additional Benefit              | Snapshots + compression reduce I/O and storage | No built-in snapshot or compression; incremental backups require extra tooling |

**Explanation:**

* In a typical LVM-thin or ext4 setup, backups often operate at the **volume level**, copying the entire 20 GB disk even if only 2 GB changed.
* ZFS breaks storage into blocks and replicates only the **changed blocks**, meaning only 2 GB of network traffic instead of 20 GB.
* Combined with **LZ4 compression**, the actual traffic might be even lower (1–1.5 GB), which is critical for a **1 GbE-constrained network**.

This concrete comparison shows why ZFS is ideal for **HA VM storage and incremental backups** in a home lab with limited network bandwidth.

---

### NAS-Side ARC Caching Improves Responsiveness

ZFS uses system RAM on the **NAS systems** as an ARC (Adaptive Replacement Cache):

* Frequently accessed (“hot”) blocks are served from memory
* Disk reads are reduced
* Latency improves for all consuming clients

> **Important:**
> ARC caching only applies where the data physically resides.
> Compute nodes accessing storage over NFS or iSCSI do **not** benefit from local ZFS caching.

---

## Primary Backup NAS

---

### Role

* Centralized backup target for:

  * Proxmox VMs
  * Proxmox containers
  * Infrastructure configuration data
* Optimized for **data protection over performance**

---

### Storage Configuration

* **Boot Device:** Dedicated **NVMe SSD**

  * Isolates OS from data
  * Simplifies recovery and upgrades
* **Backup Pool:** ZFS **RAID1 mirror**

  * Current: 2 × 3.5" HDDs
  * Prioritizes redundancy and integrity
* **Planned Expansion:**

  * May migrate to **RAIDZ1** or **RAIDZ2** as disk count increases
  * Layout decisions driven by fault tolerance, not performance

---

### Software Stack

* **Operating System:** TrueNAS (bare metal)
* **Filesystem:** ZFS
* **Backup Targets:** Proxmox VMs, containers, and infrastructure data
* **Backup Method:** Proxmox-managed backups stored in ZFS datasets

---

## High-Availability NAS

---

### Role

* Provides **shared storage** for the Proxmox cluster
* Enables VM and container placement on any compute node
* Designed for **predictable latency**, not raw throughput

---

### Storage Configuration

* **Boot Device:** Dedicated **NVMe SSD**

  * Isolates Proxmox OS from storage pools
  * Simplifies OS recovery without touching data
* **HA Storage Pool:** ZFS **RAID1 mirror** of **SATA SSDs**

  * Optimized for:

    * Low-latency random I/O
    * Consistent performance under contention
  * HDDs are intentionally avoided for this pool

> **Note:**
> HA workloads depend more on latency consistency than peak throughput.
> SATA SSDs provide predictable behavior and are well-matched to a 1 GbE network.

---

### Software Stack

* **Operating System:** Proxmox (bare metal)
* **Filesystem:** ZFS
* **Storage Exposure:** Network-backed shared storage (e.g., NFS)
* **Consumers:** Proxmox VMs and containers running on compute nodes

---

## Additional / Future Use Cases

* Self-hosted knowledge management (Evernote-like)
* File storage and synchronization services
* Internal tooling and infrastructure services

> **Network Considerations:**
> NAS systems currently reside on a **dedicated storage/backup VLAN**.
> Additional services may require firewall and routing changes to preserve isolation.

---

## Design Decisions

* **ZFS on NAS Only:** Storage intelligence lives where the disks are
* **NVMe for Boot:** Fast, isolated OS disks simplify recovery
* **SATA for Data Pools:** Cost-effective, predictable performance
* **RAID1 First:** Minimal disk count with immediate redundancy
* **Gradual Expansion:** Capacity added only when needed
* **ECC Memory:** Not used due to home lab constraints; required in enterprise environments

---

## Notes

* Data integrity and recoverability take priority over raw performance
* Backup reliability is favored over convenience
* Storage design intentionally aligns with 1 GbE network constraints
