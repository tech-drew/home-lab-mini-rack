# Proxmox Cluster Setup Guide

This guide explains how to set up a **Proxmox VE cluster** for a homelab environment with multiple nodes. It covers creating the cluster, configuring host files, adding nodes, and checking cluster health.

---

## 1. Create the Cluster on the Master Node

On the designated **master node** (e.g., `node1`), run:

```bash
pvecm create homelab-cluster
````

This command will:

* Generate a **Corosync authentication key**.
* Create the cluster configuration in `/etc/pve/corosync.conf`.
* Start Corosync and enable the cluster filesystem.

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

* Ensure that each node can **ping every other node by hostname**. Example:

```bash
ping node1.homelab.local
ping node2.homelab.local
```

* Using a `.local` domain is common, but for larger homelabs, consider `.lab` or `.home` to avoid mDNS conflicts.

---

## 3. Add Additional Nodes to the Cluster

SSH into each **additional node** and run the `pvecm add` command to join the cluster. Use either the **master node hostname** or IP address.

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

* You will be prompted for the **root password** of the master node.
* Make sure the **Proxmox web API port (8006)** is accessible from each node.
* If DNS issues occur, use the IP of the master node instead:

```bash
pvecm add 192.168.88.31
```

---

## 4. Verify Cluster Health

After adding all nodes, check the cluster status:

```bash
# Show general cluster information
pvecm status

# List all nodes in the cluster
pvecm nodes

# Check the Proxmox cluster service
systemctl status pve-cluster
```
## 5. Setup Proxmox with the non-enterprise repo. 

The enterprise repo requires a license. For the purpose of using a home-lab the enterprise features are not needed. But regardless it is strongly recommended to support the creatures of Proxmox.

The correct repos were setup by using the Proxmox PVE Post Install script found here. https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install

- The enterprise repos were disabled.
- The PVE No Subscription Repo was enabled.
- The test repos were disabled
- High Availability was left enabled since this is a cluster. I don't know if I will use any high availability features at this time.
- The notification to active a license was disabled.

## 6. Setup Proxmox Backups

Proxmox allows you to automatically back up virtual machines and snapshots on a schedule you define. If you have a NAS (Network Attached Storage), Proxmox can be configured to back up to it. However, as of December 27, 2025, I do not have a NAS. For now, I am using **local backups**. I plan to set up a NAS in the future once I can afford one. Currently, **budget constraints** are preventing me from purchasing a NAS.

I followed this guide to set up my backups: [Proxmox Backup Setup Guide](https://www.youtube.com/watch?v=lFzWDJcRsqo).

### Current Backup Settings

- **Backup Schedule:** Daily at 00:00
- **Retention Policy:** Keep 1 backup

**Note:** Each Dell Wyse 5070 Thin Client has a 256GB NVMe SSD. At the moment, this storage is used for Proxmox, virtual machines, containers, and backups. Due to the limited storage capacity, the retention policy is set to keep only one backup at a time. This setup is a "jury-rigged" solution using available equipment to build the homelab, and for now, I am making the best of the limited resources.

## 7. Add ISOs to each node

Here are the directions to add a ISO to a node locally. Best Practice is to use a NAS. If you don't have a NAS, you can add the ISO onto each node.

- Click on a Node>Local>Iso Images>Upload an Iso.
- Kubuntu 24.0.3 will be used as a starter image. Ubuntu has a bunch of community support. However, I don't like gnome, so I will use Kubuntu so I have access to the KDE Plasma GUI whenever a GUI is necessary.
- A lot of VM management can be done via SSH. So I don't expect to heavily rely on GUIs.
## 8. Proxmox Hepler Scripts

The following website can be used to get a bunch of useful scripts to help automate proxmox work. In particular, I found the `pve post install` script to be very helpful.

https://community-scripts.github.io/ProxmoxVE/scripts
