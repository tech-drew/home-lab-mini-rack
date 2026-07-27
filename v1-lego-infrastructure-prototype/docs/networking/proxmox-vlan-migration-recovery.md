# Proxmox Cluster VLAN Migration & Recovery Log

## Overview
Originally, this Proxmox cluster operated on the `192.168.88.0/24` subnet. After migrating the management traffic to **VLAN 99** (`10.100.99.0/24`), cluster connectivity was lost. Although the physical interfaces were moved to the new VLAN, the nodes remained configured with their legacy IP addresses, resulting in a complete loss of network access.

To restore connectivity, I accessed each node locally using a physical monitor and keyboard. I manually updated the network configuration in `/etc/network/interfaces` and the local resolution in `/etc/hosts`. After verifying the new IPs with `ip a`, I rebooted the nodes and confirmed external connectivity by successfully pinging `8.8.8.8`. (Note: DNS is not configured for this subnet, so pings to external IPs must be used for verification).

### Migration IP Mapping
| Node Name | Old IP Address | New IP Address |
| :--- | :--- | :--- |
| node1 | 192.168.88.10 | 10.100.99.11 |
| node2 | 192.168.88.20 | 10.100.99.12 |
| node3 | 192.168.88.30 | 10.100.99.13 |
| node4 | 192.168.88.40 | 10.100.99.14 |

---

## 1. Network Environment
* **Router:** MikroTik RB5009
* **Management VLAN:** 99
* **Management Subnet:** `10.100.99.0/24`
* **Gateway:** `10.100.99.1`

## 2. Configuration Files

### /etc/hosts (Applied to all nodes)
```bash
127.0.0.1       localhost
10.100.99.11    node1
10.100.99.12    node2
10.100.99.13    node3
10.100.99.14    node4

```

### /etc/network/interfaces (Example for Node 1)

```bash
auto lo
iface lo inet loopback

iface enp1s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.100.99.11/24
        gateway 10.100.99.1
        bridge-ports enp1s0
        bridge-stp off
        bridge-fd 0

```

---

## 3. Cluster Recovery (Corosync)

Once basic networking was restored, the Proxmox Web GUI indicated the cluster was broken. This was due to `corosync.conf` still referencing the old `192.168.88.x` IPs. Because the cluster lacked quorum, the `/etc/pve` directory was read-only, requiring a manual override to update the configuration.

### /etc/pve/corosync.conf (Current Version: 5)

```bash
logging {
  debug: off
  to_syslog: yes
}

nodelist {
  node {
    name: node1
    nodeid: 1
    quorum_votes: 1
    ring0_addr: 10.100.99.11
  }
  node {
    name: node2
    nodeid: 2
    quorum_votes: 1
    ring0_addr: 10.100.99.12
  }
  node {
    name: node3
    nodeid: 3
    quorum_votes: 1
    ring0_addr: 10.100.99.13
  }
  node {
    name: node4
    nodeid: 4
    quorum_votes: 1
    ring0_addr: 10.100.99.14
  }
}

quorum {
  provider: corosync_votequorum
}

totem {
  cluster_name: homelab-cluster
  config_version: 5
  interface {
    linknumber: 0
  }
  ip_version: ipv4-6
  link_mode: passive
  secauth: on
  version: 2
}

```

---

## 4. Recovery Procedure (If Quorum is Lost)

If the nodes cannot communicate and the configuration filesystem is locked, execute the following steps on **each node**:

1. **Stop Cluster Services:** `systemctl stop pve-cluster corosync`
2. **Force Local Mode:** `pmxcfs -l` (Unlocks the `/etc/pve` filesystem for manual editing).
3. **Update Configuration:** `nano /etc/pve/corosync.conf`
* **Increment `config_version`:** If the current version is `5`, change it to `6`. This must be identical on all nodes.
* **Update IPs:** Ensure all `ring0_addr` fields reflect the new `10.100.99.x` subnet.


4. **Restart Services:** `killall pmxcfs && systemctl start pve-cluster corosync`
5. **Verify Status:** `pvecm status` (Confirm that **Quorate** is **Yes** and all 4 nodes are visible).
