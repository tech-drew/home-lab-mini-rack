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

**Tips:**

* `pvecm status` shows **quorum**, node membership, and cluster configuration version.
* All nodes should be **quorate** and online.
* If a node fails to join, double-check `/etc/hosts`, network connectivity, and firewall settings.

---

## 5. Optional Best Practices

* Use **short, descriptive hostnames** like `node1`, `node2`, etc.
* Keep `/etc/hosts` **identical on all nodes**.
* Avoid special characters in hostnames.
* Plan for future expansion by numbering nodes sequentially.

```
