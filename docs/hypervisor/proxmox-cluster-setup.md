# Proxmox Cluster Setup Guide

## ZFS Design Considerations for a 1 GbE–Limited Network

Before diving into the cluster setup, it’s important to explain the filesystem choice. This Proxmox cluster is deployed in a home lab environment with a hard 1 GbE network limitation. Because network bandwidth is the primary bottleneck, the cluster must be configured to minimize the amount of data sent over the network. This is accomplished by using ZFS.

**Note:** ZFS must be selected and configured during the initial Proxmox installation. Proxmox does not support converting an existing root filesystem (for example, LVM or ext4) to ZFS after installation. Changing to ZFS later would require reinstalling Proxmox and wiping the node. This design decision should be made before installing proxmox on the first cluster node.

ZFS is a strong fit for this constraint for the following reasons:

### Incremental, Block-Level Replication

ZFS replication is snapshot-based and incremental, meaning it transfers only the blocks that have changed since the last snapshot.

Example:

- A 20 GB VM disk
- Only 1–2 GB of data changed since the last snapshot

In this scenario, ZFS replicates only the changed blocks, not the entire disk. This can reduce backup and HA-related network traffic by 90% or more compared to volume-based approaches such as LVM-thin, which often operate at the whole-volume level.

This behavior is especially beneficial on a 1 GbE network, where full-disk transfers would quickly saturate available bandwidth.

### Compression Before Network Transfer

ZFS applies fast, transparent compression (typically LZ4) before data is written to disk or sent over the network.

- VM disk data commonly compresses 1.5×–2×
- Less data traverses the network
- Work is shifted from the network (the bottleneck) to CPU, which is plentiful in this environment

This results in faster replication, backups, and reduced network congestion during normal cluster operations.

### ARC Caching Reduces Data Churn

ZFS aggressively uses system RAM as an ARC (Adaptive Replacement Cache):

- Frequently accessed (“hot”) data is served from memory
- Fewer disk writes occur
- Fewer blocks change between snapshots

Smaller change sets between snapshots translate directly into smaller replication deltas, further reducing network traffic.

### Summary

ZFS does not increase network speed—it minimizes how much data must cross the network.

In a Proxmox cluster constrained to 1 GbE:

- Backups complete faster
- Replication is less disruptive
- HA-related storage traffic is more manageable

For this home lab, ZFS is a deliberate choice to work *with* the network limitation rather than against it.


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
