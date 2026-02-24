# Proxmox Network Storage Setup

**ZFS Mirror + iSCSI Target Configuration**

This guide walks through setting up a mirrored **ZFS pool** and exporting a ZVOL over **iSCSI** on Proxmox VE.

## 1. List Available Disks (Using Persistent IDs)

Use the `/dev/disk/by-id/` directory to identify disks by serial number:

```bash
ls -l /dev/disk/by-id/
```

Example output:

```text
ata-Example_SSD_1_1234567890
ata-Example_SSD_1_0987654321
ata-Example_SSD_1_1122334455
```

Next, cross-reference with `lsblk` to confirm size and type:

```bash
lsblk -o NAME,SIZE,MODEL
```

Example:

```text
sda   2T  Example SSD 1
sdb   2T  Example SSD 1
sdc   1T  Example SSD 1
```
Identify the disks you want to use for your ZFS pool. Using **persistent device IDs** from `/dev/disk/by-id/` ensures that ZFS references the correct disks, even if device names change after a reboot or if you move drives to different SATA ports.

For example, if you use a command like:

```bash
zpool create -o ashift=12 vmdata mirror /dev/sda /dev/sdc
```

it will work initially. However, the `/dev/sdX` identifiers are dynamically assigned by Linux at boot. This means that after a reboot or if the drives are connected to different SATA ports, the same disks could be assigned new device names. For instance, a disk that was `/dev/sda` before reboot could appear as `/dev/sdd` afterward. Using `/dev/disk/by-id/...` prevents this problem and makes your pool creation safer and more reliable.

---

## 2. Wipe the Disks

**Warning:** This will permanently erase all data on the selected disks.

```bash
wipefs -a /dev/sda
wipefs -a /dev/sdc
```

This ensures no old filesystem or RAID metadata interferes with ZFS.

**Note:** It is okay to wipe the disks using the dynamically assigned identifier even though step 3 will use the persistent disk IDs.

---

## 3. Create the ZFS Mirror Pool

Create a mirrored ZFS pool named `vmdata` using the persistent disk IDs and enable **LZ4 compression**:

```bash
zpool create -o ashift=12 \
  -O compression=lz4 \
  vmdata mirror \
  /dev/disk/by-id/ata-Example_SSD_1_1234567890 \
  /dev/disk/by-id/ata-Example_SSD_1_0987654321
```

### Explanation of Options

* `-o ashift=12` → Aligns ZFS to 4K sector drives for optimal performance.
* `-O compression=lz4` → Enables LZ4 compression at the pool level. LZ4 is fast, efficient, and reduces storage usage for general workloads. All datasets and ZVOLs in this pool inherit this compression unless overridden.
* `mirror` → In ZFS, a mirror provides RAID1-style redundancy. Unlike traditional RAID1, ZFS integrates end-to-end checksumming and self-healing, so it can detect and repair silent data corruption at the filesystem level. Traditional RAID1 mirrors blocks but cannot detect corruption beyond disk ECC.
* Disk IDs (`/dev/disk/by-id/...`) → Persistent identifiers tied to the physical disk, ensuring the pool remains consistent across reboots or SATA port changes.

### Verify Pool Status

```bash
zpool status
```

You should see the mirrored vdev online, with all disks healthy and ready for use.

---

## 4. Configure Recommended ZFS Settings

Apply performance and optimization settings:

```bash
zfs set atime=off vmdata
zfs set xattr=sa vmdata
```

**Explanation:**
https://www.youtube.com/watch?v=n41enLMUrgU
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

**Note:** This process was repeated for NVME SSDs in the NAS to create a second smaller but faster storage pool.


