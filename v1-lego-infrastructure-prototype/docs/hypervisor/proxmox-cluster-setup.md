# Proxmox Cluster Setup Guide

## Note on Storage

This Proxmox cluster uses **network-attached storage (NAS)** for VM and container disks. Detailed information about the storage setup, including ZFS configuration, replication, and pool design, can be found in the [NAS Setup / Storage Documentation](./storage.md).

---

## 0. Networking Considerations

Before creating the cluster, ensure your Proxmox nodes are configured for the VLAN-aware network as documented in [Proxmox Network Configuration](../networking/proxmox-network.md).

* The network adapter used for all VLAN traffic is named **eth0** on each node.
* The management IP for each node resides on VLAN 99. For example, `10.100.99.11` is the management IP for **node1**. Adjust for your node and VLAN.
* This setup is necessary because the router ports only allow **tagged traffic**, and untagged frames are blocked.
* Using a VLAN-aware bridge (`vmbr0`) ensures all VLANs (20, 30, 40, 99, etc.) are accessible on a single NIC, simplifying management and VM connectivity.

> **Tip:** Always confirm your NIC names and VLAN IPs match the configuration described in `proxmox-network.md` before proceeding.

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

# Homelab nodes (replace with VLAN 99 IPs from your network config)
10.100.99.11 node1.homelab.local node1
10.100.99.12 node2.homelab.local node2
10.100.99.13 node3.homelab.local node3
10.100.99.14 node4.homelab.local node4

# IPv6 configuration (optional)
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
* IPs should correspond to the management VLAN (VLAN 99) defined in your network documentation.

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
* If DNS issues occur, use the management VLAN IP instead:

  ```bash
  pvecm add 10.100.99.11
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

---

**Key takeaway:** Always verify that the network interface configuration on each node matches the VLAN-aware setup in [proxmox-network.md](../networking/proxmox-network.md). This ensures that all nodes can communicate over VLAN 99 for management and correctly handle tagged traffic for other VLANs.


