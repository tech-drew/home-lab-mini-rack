# Proxmox Cluster Setup Guide

## Note on Storage

This Proxmox cluster uses **network-attached storage (NAS)** for VM and container disks. Detailed information about the storage setup, including ZFS configuration, replication, and pool design, can be found in the [NAS Setup / Storage Documentation](./storage.md).

---

## 1. Create the Cluster on the Master Node

On the designated **master node** (e.g., `node1`), run:

```bash
pvecm create homelab-cluster
```

This command will:

* Generate a **Corosync authentication key**
* Create the cluster configuration in `/etc/pve/corosync.conf`
* Start Corosync and enable the cluster filesystem

After this step, your master node will be the first member of the cluster.

---

## 2. Configure `/etc/hosts` on All Nodes

SSH into each node to ensure that all nodes have the **same `/etc/hosts` file**. This ensures proper hostname resolution for the cluster.

```bash
ssh root@node1
ssh root@node2
ssh root@node3
ssh root@node4
```

Edit `/etc/hosts` on each node as follows:

```text
127.0.0.1 localhost.localdomain localhost

# Homelab nodes
192.168.88.31 node1.homelab.local node1
192.168.88.32 node2.homelab.local node2
192.168.88.33 node3.homelab.local node3
192.168.88.34 node4.homelab.local node4

# IPv6 configuration (recommended for IPv6-capable hosts)
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

**Notes:**

* Ensure that each node can **ping every other node by hostname**
* Using a `.local` domain is common, but for larger homelabs consider `.lab` or `.home` to avoid mDNS conflicts

---

## 3. Add Additional Nodes to the Cluster

SSH into each **additional node** and run the `pvecm add` command to join the cluster.

```bash
# Node 2
ssh root@node2
pvecm add node1.homelab.local

# Node 3
ssh root@node3
pvecm add node1.homelab.local

# Node 4
ssh root@node4
pvecm add node1.homelab.local
```

**Notes:**

* You will be prompted for the **root password** of the master node
* Ensure **port 8006** is reachable
* If DNS issues occur, use the IP instead:

  ```bash
  pvecm add 192.168.88.31
  ```

---

## 4. Verify Cluster Health

```bash
pvecm status
pvecm nodes
systemctl status pve-cluster
```

---

## 5. Setup Proxmox with the Non-Enterprise Repo

The enterprise repository requires a license. For a home lab, the enterprise features are not required, though it is strongly recommended to support the creators of Proxmox if possible.

Repos were configured using the **Proxmox PVE Post Install Script**:
[https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install](https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install)

* Enterprise repo disabled
* PVE No-Subscription repo enabled
* Test repo disabled
* High Availability left enabled (clustered environment)
* License notification disabled

---

## 6. Setup Proxmox Backups

Currently, no NAS is available due to budget constraints, so **local backups** are used.

Guide followed:
[Proxmox Backup Setup Guide](https://www.youtube.com/watch?v=lFzWDJcRsqo)

### Current Backup Settings

* **Schedule:** Daily at 00:00
* **Retention:** Keep 1 backup

Each Dell Wyse 5070 Thin Client has a **256 GB NVMe SSD**, which is shared by:

* Proxmox
* VMs
* Containers
* Backups

This is a temporary, resource-constrained setup using available hardware.

---

## 7. Add ISOs to Each Node

Without shared storage, ISOs must be uploaded per-node.

Steps:

* Node → Local → ISO Images → Upload ISO
* **Kubuntu 24.0.3** is used as the base image
* Ubuntu ecosystem provides strong community support
* KDE Plasma is preferred over GNOME
* Most VM management is done via SSH

---

## 8. Proxmox Helper Scripts

Helpful automation scripts can be found here:

[https://community-scripts.github.io/ProxmoxVE/scripts](https://community-scripts.github.io/ProxmoxVE/scripts)

The **Post PVE Install** script was especially useful for initial setup.
