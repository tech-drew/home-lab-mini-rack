**ZFS Mirror + iSCSI Target Configuration**

This guide walks through setting up a mirrored **ZFS pool** and exporting a ZVOL over **iSCSI** on Proxmox VE, including how to configure clean, enterprise-style disk names in `zpool status`.

---

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

Using **persistent device IDs** from `/dev/disk/by-id/` ensures ZFS references the correct disks even if Linux device names change after reboot.

---

## 2. (Optional but Recommended) Configure Clean Disk Names

By default, `zpool status` will show long disk IDs like:

```
ata-Example_SSD_1_1234567890
```

To make the output clean and enterprise-style (e.g., `disk1`, `disk2`), configure a ZFS vdev alias file.

### 2.1 Create the VDEV Mapping File

Create the configuration file:

```bash
nano /etc/zfs/vdev_id.conf
```

Add aliases using the same disk IDs from Step 1:

```
alias disk1 /dev/disk/by-id/ata-Example_SSD_1_1234567890
alias disk2 /dev/disk/by-id/ata-Example_SSD_1_0987654321
```

Rules:

* No spaces in alias names
* Use underscores if needed
* Names are cosmetic but persistent

You may alternatively use descriptive names:

```
alias 2TB_SATA_SSD_disk1 /dev/disk/by-id/ata-Example_SSD_1_1234567890
alias 2TB_SATA_SSD_disk2 /dev/disk/by-id/ata-Example_SSD_1_0987654321
```

---

## 3. Wipe the Disks

**Warning:** This permanently erases all data.

```bash
wipefs -a /dev/sda
wipefs -a /dev/sdc
```

It is acceptable to wipe using `/dev/sdX` identifiers. The pool will be created using persistent IDs.

---

## 4. Create the ZFS Mirror Pool

Create a mirrored ZFS pool named `vmdata` using the persistent disk IDs:

```bash
zpool create -o ashift=12 \
  -O compression=lz4 \
  vmdata mirror \
  /dev/disk/by-id/ata-Example_SSD_1_1234567890 \
  /dev/disk/by-id/ata-Example_SSD_1_0987654321
```

### Explanation of Options

* `-o ashift=12` → Aligns to 4K sectors
* `-O compression=lz4` → Enables LZ4 compression
* `mirror` → RAID1-style redundancy with ZFS self-healing
* Persistent IDs ensure stability across reboots

---

## 5. Export and Re-Import to Activate Custom Names

If you configured `/etc/zfs/vdev_id.conf`, export and re-import the pool to activate aliases.

### Export:

```bash
zpool export vmdata
```

### Re-import (recommended explicit method):

```bash
zpool import -d /dev/disk/by-id -d /dev/disk/by-vdev vmdata
```

Or simply:

```bash
zpool import vmdata
```

ZFS will automatically generate:

```
/dev/disk/by-vdev/disk1
/dev/disk/by-vdev/disk2
```

---

## 6. Verify Clean Output

```bash
zpool status
```

You should now see:

```
mirror-0
  disk1
  disk2
```

Instead of long ATA identifiers.

### Important Notes

* This is cosmetic only
* It does not change pool structure
* It does not rewrite metadata
* It survives reboot
* Safe for iSCSI-backed storage

If `vdev_id.conf` is removed, ZFS will revert to full disk IDs.

---

## 7. Configure Recommended ZFS Settings

```bash
zfs set atime=off vmdata
zfs set xattr=sa vmdata
```

* `atime=off` reduces write overhead
* `xattr=sa` improves extended attribute handling

---

## 8. Create a ZVOL for iSCSI

Create a 1.5 TB block device:

```bash
zfs create -s -V 1.5T -o volblocksize=16K vmdata/pve-iscsi
```

* `-V 1.5T` creates a ZVOL at `/dev/zvol/vmdata/pve-iscsi`
* `volblocksize=16K` optimized for VM workloads

Leave 10–25% pool capacity free for performance stability.

### Verify:

```bash
zfs list
```

---

## 9. Configure Sync Behavior

```bash
zfs set sync=standard vmdata/pve-iscsi
```

Avoid `sync=disabled` unless you fully understand the risks.

---

## 10. Install iSCSI Target Tools

```bash
apt install targetcli-fb
```

---

## 11. Load Required Kernel Modules

```bash
modprobe target_core_user
modprobe iscsi_target_mod
```

Verify:

```bash
lsmod | grep iscsi
```

---

## 12. Configure the iSCSI Target

Launch the iSCSI configuration shell:

```bash
targetcli
```

---

# 12.1 Create Backstore

Create a block backstore using the ZFS ZVOL created earlier.

```
/> backstores/block create pve-iscsi /dev/zvol/vmdata/pve-iscsi
```

Verify the backstore:

```
/> ls /backstores/block
```

Expected output:

```
o- pve-iscsi
```

---

# 12.2 Create the iSCSI Target

Create the iSCSI target for the storage server.

```
/> iscsi/ create iqn.2026-03.homelab.local:pve-storage1
```

Verify:

```
/> ls /iscsi
```

Expected output:

```
o- iqn.2026-03.homelab.local:pve-storage1
```

---

# 12.3 Create the LUN

Expose the ZVOL to the target as a LUN.

```
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/luns create /backstores/block/pve-iscsi
```

Verify:

```
/> ls iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/luns
```

Expected output:

```
o- lun0 -> /backstores/block/pve-iscsi
```

---

# 12.4 Add Portal (Storage Network)

Bind the iSCSI target to the storage VLAN interface.

```
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/portals create 10.100.60.15
```

Verify:

```
/> ls iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/portals
```

Expected output:

```
o- 10.100.60.15:3260
```

---

## Storage Network Configuration

Before creating the portal, ensure that the host system is actually assigned the IP address specified in the command. The iSCSI service can only bind to an IP address that exists on one of the host’s network interfaces.

Network design:

| VLAN    | Network        | Purpose                 |
| ------- | -------------- | ----------------------- |
| VLAN 60 | 10.100.60.0/24 | Storage traffic (iSCSI) |
| VLAN 99 | 10.100.99.0/24 | Management traffic      |

Storage server addresses:

| Host         | Storage IP   | Management IP |
| ------------ | ------------ | ------------- |
| pve-storage1 | 10.100.60.15 | 10.100.99.15  |

The storage host is configured with the following interfaces:

```bash
auto lo
iface lo inet loopback

# Physical NIC
auto eth0
iface eth0 inet manual
    mtu 9000

# VLAN-aware bridge
auto vmbr0
iface vmbr0 inet manual
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
    mtu 9000

# Storage VLAN
auto vmbr0.60
iface vmbr0.60 inet static
    address 10.100.60.15/24
    mtu 9000

# Management VLAN
auto vmbr0.99
iface vmbr0.99 inet static
    address 10.100.99.15/24
    gateway 10.100.99.1
    mtu 1500

source /etc/network/interfaces.d/*
```

---

# 12.5 Create ACLs for Proxmox Nodes

Access Control Lists (ACLs) define which iSCSI initiators are allowed to connect to the storage target.

Your Proxmox cluster nodes use the following initiator names:

| Node  | Initiator IQN                   |
| ----- | ------------------------------- |
| node1 | iqn.2026-03.homelab.local:node1 |
| node2 | iqn.2026-03.homelab.local:node2 |
| node3 | iqn.2026-03.homelab.local:node3 |
| node4 | iqn.2026-03.homelab.local:node4 |

Create ACL entries for each node:

```
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/acls create iqn.2026-03.homelab.local:node1
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/acls create iqn.2026-03.homelab.local:node2
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/acls create iqn.2026-03.homelab.local:node3
/> iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/acls create iqn.2026-03.homelab.local:node4
```

Verify ACL entries:

```
/> ls iscsi/iqn.2026-03.homelab.local:pve-storage1/tpg1/acls
```

Expected output:

```
o- iqn.2026-03.homelab.local:node1
o- iqn.2026-03.homelab.local:node2
o- iqn.2026-03.homelab.local:node3
o- iqn.2026-03.homelab.local:node4
```

---

# 12.6 Save Configuration

Persist the configuration so it survives reboots.

```
/> saveconfig
```

Exit the shell:

```
/> exit
```

The configuration is stored at:

```
/etc/rtslib-fb-target/saveconfig.json
```

---

# 12.7 Final Verification

Run:

```bash
targetcli ls
```

You should see a structure similar to:

```
o- backstores
| o- block
|   o- pve-iscsi
|
o- iscsi
  o- iqn.2026-03.homelab.local:pve-storage1
    o- tpg1
      o- luns
      | o- lun0 -> /backstores/block/pve-iscsi
      o- portals
      | o- 10.100.60.15:3260
      o- acls
        o- iqn.2026-03.homelab.local:node1
        o- iqn.2026-03.homelab.local:node2
        o- iqn.2026-03.homelab.local:node3
        o- iqn.2026-03.homelab.local:node4
```

---

# Result

The ZFS ZVOL is now exported as an iSCSI target and restricted to the four Proxmox cluster nodes.

## Verify connectivity on each of the notes

**Note:** The error for node1 was all the ports were not tagged to allow vlan 60 on the router. Don't forget to check your networking!

```bash
iscsiadm -m discovery -t st -p 10.100.60.15
```

<img width="2561" height="1601" alt="image" src="https://github.com/user-attachments/assets/4eae4ddf-3935-4856-abc7-5e3733646657" />

## Login to the network storage from each node

```bash
iscsiadm -m node -T iqn.2026-03.homelab.local:pve-storage1 -p 10.100.60.15 --login
```

<img width="2561" height="1601" alt="image" src="https://github.com/user-attachments/assets/ca810640-217c-4ec6-87db-8f57e4345d76" />

## Verify the session

```bash
iscsiadm -m session
```
<img width="2561" height="1601" alt="image" src="https://github.com/user-attachments/assets/bb9920f8-4ec5-4f5b-95ae-296a2839df44" />

## Use lsblk to verify the disk

```bash
lsblk
```
<img width="2561" height="1601" alt="image" src="https://github.com/user-attachments/assets/96efb86e-5628-4372-b722-4969d814da52" />

## Add the Disk to Proxmox to be Usable by the Proxmox Cluster

<img width="2561" height="1601" alt="image" src="https://github.com/user-attachments/assets/1e131807-e122-4553-95bb-0c68bcce6de9" />




