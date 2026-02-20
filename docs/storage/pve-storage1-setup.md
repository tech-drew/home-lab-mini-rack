lsblk to list disks

<img width="798" height="635" alt="image" src="https://github.com/user-attachments/assets/d0733c2e-7beb-4777-b35f-76eddaf4e36c" />

Wipe the disks we will use in the storage pool

wipefs -a /dev/sda
wipefs -a /dev/sdc

Create the pool
zpool create -o ashift=12 vmdata mirror /dev/sda /dev/sdc

Verify the pool status
zpool status

Configure zfs settings
zfs set compression=lz4 vmdata
zfs set atime=off vmdata
zfs set xattr=sa vmdata

create zvol for iscsi
zfs create -V 1.5T -o volblocksize=16K vmdata/pve-iscsi

verify the zfs zvol is created
zfs list

Explicitly setting up sync for iscsi
zfs set sync=standard vmdata/pve-iscsi

Get the tooling for the iscsi target setup
apt install targetcli-fb

Make sure the kernel modules are loaded
modprobe target_core_user
modprobe iscsi_target_mod

configure iscsi
targetcli
/> backstores/block create pve-iscsi /dev/zvol/vmdata/pve-iscsi
/> iscsi/ create iqn.2026-02.com.example:pve-storage1
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/luns create /backstores/block/pve-iscsi
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/portals create 10.100.60.10
/> iscsi/iqn.2026-02.com.example:pve-storage1/tpg1/acls create iqn.2026-02.com.example:pve-node1
/> saveconfig
/> exit




