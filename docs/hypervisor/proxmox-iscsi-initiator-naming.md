# Proxmox iSCSI Initiator Naming (Cluster Nodes)

This guide explains how to **view and update the iSCSI initiator names** on Proxmox nodes so they are clean and consistent with your homelab naming scheme.

Using readable IQNs makes ACL management much easier when configuring shared storage.

---

# 1. Check the Current iSCSI Initiator Name

Run the following command on each Proxmox node.

```bash
cat /etc/iscsi/initiatorname.iscsi
```

Example output:

```
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
InitiatorName=iqn.1993-08.org.debian:01:33dbfece5c9d
```

The value after `InitiatorName=` is the node's **iSCSI initiator identifier**.

---

# 2. Recommended Naming Scheme

For a homelab cluster with nodes:

```
node1
node2
node3
node4
```

Use the following consistent format:

```
iqn.2026-03.homelab.local:node1
iqn.2026-03.homelab.local:node2
iqn.2026-03.homelab.local:node3
iqn.2026-03.homelab.local:node4
```

Structure of an IQN:

```
iqn.<year>-<month>.<domain>:<hostname>
```

Example:

```
iqn.2026-03.homelab.local:node1
```

---

# 3. Update the Initiator Name

Edit the initiator configuration file.

```bash
nano /etc/iscsi/initiatorname.iscsi
```

Replace the existing value.

Example change:

**Before**

```
InitiatorName=iqn.1993-08.org.debian:01:33dbfece5c9d
```

**After**

```
InitiatorName=iqn.2026-03.homelab.local:node1
```

Repeat on each node with the appropriate hostname.

Example table:

| Node  | InitiatorName                   |
| ----- | ------------------------------- |
| node1 | iqn.2026-03.homelab.local:node1 |
| node2 | iqn.2026-03.homelab.local:node2 |
| node3 | iqn.2026-03.homelab.local:node3 |
| node4 | iqn.2026-03.homelab.local:node4 |

---

# 4. Restart the iSCSI Service

After modifying the file, restart the iSCSI daemon.

```bash
systemctl restart iscsid
```

---

# 5. Verify the Change

Confirm the new initiator name is active.

```bash
cat /etc/iscsi/initiatorname.iscsi
```

Expected example:

```
InitiatorName=iqn.2026-03.homelab.local:node1
```

---

# 6. Verify From the Storage Server

On the iSCSI storage host (`pve-storage1`), open the target configuration:

```bash
targetcli
```

List configuration:

```
ls
```

Expected ACL entries after configuration:

```
acls
  o- iqn.2026-03.homelab.local:node1
  o- iqn.2026-03.homelab.local:node2
  o- iqn.2026-03.homelab.local:node3
  o- iqn.2026-03.homelab.local:node4
```

---

# 7. Important Notes

* Initiator names **must be unique** for each node.
* Changing the initiator name is safe **before connecting to iSCSI storage**.
* Once storage is in use, changing the name requires reconnecting the iSCSI session.

---

# 8. Summary

Steps performed:

1. Checked existing initiator name
2. Updated it to match node hostname
3. Restarted `iscsid`
4. Verified configuration
5. Prepared nodes for ACL configuration on the storage server

This keeps the cluster configuration **clean, readable, and easy to manage** when working with shared iSCSI storage.
