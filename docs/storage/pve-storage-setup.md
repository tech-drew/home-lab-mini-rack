# Proxmox Network Storage Setup

**ZFS Mirror + iSCSI Target Configuration**

This guide walks through setting up a mirrored **ZFS pool** and exporting a ZVOL over **iSCSI** on Proxmox VE.

---

## 1. List Available Disks

Use `lsblk` to identify available disks:

```bash
lsblk
```

<img width="798" height="635" alt="image" src="https://github.com/user-attachments/assets/d0733c2e-7beb-4777-b35f-76eddaf4e36c" />

Confirm the disks you plan to use (for example: `/dev/sda` and `/dev/sdc` are the drives I am using).

---

## 2. Wipe the Disks

**Warning:** This will permanently erase all data on the selected disks.

```bash
wipefs -a /dev/sda
wipefs -a /dev/sdc
```

---

## 3. Create the ZFS Mirror Pool

Create a mirrored ZFS pool named `vmdata`:

```bash
zpool create -o ashift=12 vmdata mirror /dev/sda /dev/sdc
```

* `ashift=12` ensures proper alignment for 4K sector drives.
* `mirror` creates RAID1-style redundancy.

### Verify Pool Status

```bash
zpool status
```

---

## 4. Configure Recommended ZFS Settings

Apply performance and optimization settings:

```bash
zfs set compression=lz4 vmdata
zfs set atime=off vmdata
zfs set xattr=sa vmdata
```

**Explanation:**

* `compression=lz4` improves performance and saves space.
* `atime=off` disables access time updates, reducing write overhead.
* `xattr=sa` stores extended attributes more efficiently.

---

## 5. Create a ZVOL for iSCSI

Create a 1.5 TB block device (ZVOL) for use with iSCSI.

This configuration uses two 2 TB SATA SSDs in a mirrored vdev. In ZFS, performance degrades significantly as pool utilization increases, particularly beyond ~80%. For this reason, it is best practice to keep at least 10–20% of the pool capacity free to reduce fragmentation and maintain write performance.

To provide additional headroom for performance consistency and metadata overhead, approximately 25% of the pool capacity is intentionally left unallocated.

```bash
zfs create -V 1.5T -o volblocksize=16K vmdata/pve-iscsi
```

* `-V 1.5T` creates a 1.5 TB ZFS volume (ZVOL), which presents as a block device at `/dev/zvol/vmdata/pve-iscsi`.
* `volblocksize=16K` sets the ZVOL block size to 16 KB. This value should generally align with the expected workload. A 16 KB block size is commonly used for VM storage and works well for mixed workloads, though 8 KB may be preferable for database-heavy workloads, and larger sizes (e.g., 32–64 KB) may benefit sequential workloads.

Ensure the selected `volblocksize` matches your anticipated I/O profile, as it cannot be changed after the ZVOL is created.

### Verify the ZVOL

```bash
zfs list
```

---

## 6. Configure Sync Behavior for iSCSI

```bash
zfs set sync=standard vmdata/pve-iscsi
```

* `sync=standard` ensures safe write behavior.
* Avoid `sync=disabled` unless you fully understand the data-loss risks.

---

## 7. Install iSCSI Target Tools

Install the Linux LIO target framework tools:

```bash
apt install targetcli-fb
```

---

## 8. Load Required Kernel Modules

```bash
modprobe target_core_user
modprobe iscsi_target_mod
```

To verify:

```bash
lsmod | grep iscsi
```

---

## 9. Configure the iSCSI Target

Launch the configuration shell:

```bash
targetcli
```

### Create the Backstore

```
/> backstores/block create pve-iscsi /dev/zvol/vmdata/pve-iscsi
```

### Create the iSCSI Target

```
/> iscsi/ create iqn.2026-02.com.example:pve-storage1
```

### Create a LUN

```
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/luns create /backstores/block/pve-iscsi
```

### Add a Network Portal

Replace with your storage server IP address:

```
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/portals create 10.100.60.10
```

### Create an ACL for a Proxmox Node

```
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/acls create iqn.2026-02.com.example:pve-node1
```

### Save and Exit

```
/> saveconfig
/> exit
```

---

# Summary

You have:

* Created a mirrored ZFS pool
* Applied recommended performance tuning
* Created a ZVOL for block storage
* Exported the volume via iSCSI
* Configured access control for a Proxmox node

The storage can now be added in **Datacenter → Storage → Add → iSCSI** within Proxmox.
