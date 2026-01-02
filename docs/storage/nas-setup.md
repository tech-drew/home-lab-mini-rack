# NAS Setup Documentation

## Overview

This NAS is used primarily for backing up Proxmox virtual machines, containers, and related infrastructure.  
It runs **TrueNAS natively (bare metal)** and is designed for reliability, data integrity, and gradual expansion as storage needs grow.

---

## Hardware Configuration

### Chassis & Slots
- **NVMe Slots:** 2
- **3.5" HDD Bays:** 4

### Current Disk Usage

#### NVMe
- Slot 1: TrueNAS OS boot device
- Slot 2: Unused (reserved for future expansion or cache)

#### HDDs
- Bay 1 & 2: HDDs in RAID1 mirror for backup storage
- Bay 3 & 4: Empty (planned for future expansion)

### Memory
- **ECC:** Not used  
- Decision based on professional guidance and home lab context; budget also factored in.  
- In an enterprise environment, ECC would be standard.

---

## Storage Configuration

### Boot Device
- Dedicated NVMe drive for TrueNAS OS  
- Keeps OS separate from data for easier recovery and upgrades
- TrueNAS does not perform well on eMMC drives. It is recommended to install TrueNAS on an NVME or SATA SSD.

### Backup Storage Pool
- **Current Pool Type:** RAID1 (Mirror) with 2 × 3.5" HDDs  
- Prioritizes redundancy and data integrity; performance is secondary

### Planned Expansion
- Remaining HDD bays will be populated as needed  
- Storage may be migrated to **RAIDZ1** (single parity) or **RAIDZ2** (dual parity) depending on total disk count and desired fault tolerance

---

## Software Stack

- **Operating System:** TrueNAS (bare metal)  
- **Backup Sources:** Proxmox VMs and containers, infrastructure data  
- Backups managed from Proxmox, stored on ZFS datasets

---

## Additional / Future Use Cases

Potential future services include:  
- **Media Server:** Self-hosted Jellyfin  
- **Notes / Knowledge Management:** Evernote-like self-hosted application  
- **File Storage / Sync:** Self-hosted file storage similar to Google Drive / Docs  

> Network Considerations: NAS is currently on a **dedicated backups VLAN**.  
> Running user-facing services may require additional VLANs, firewall adjustments, and careful access control to maintain backup isolation.

---

## Design Decisions

- **TrueNAS Bare Metal:** Direct hardware access, ZFS integrity, lower complexity  
- **RAID1 Start:** Minimal disk requirement, simple redundancy  
- **Gradual Expansion:** Adds storage only as needed, keeps costs manageable  
- **ECC Decision:** Not used due to cost and home lab context; acceptable risk in this environment, but enterprise setups would use ECC

---

## Notes

- Primary focus: **data protection and backups**  
- Additional services must not compromise isolation or reliability  
- Restore integrity is prioritized over raw performance
