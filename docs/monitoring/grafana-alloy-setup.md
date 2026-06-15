# Grafana Alloy Log Collection Setup

This guide explains how to configure **Grafana Alloy** on Proxmox nodes, PBS, and storage systems to send logs to a central **Loki instance** at:

```

[http://10.100.10.18:3100](http://10.100.10.18:3100)

````

The goal is a simple, lightweight, and centralized logging setup with minimal per-node configuration.

---

# Architecture

```text
Proxmox / PBS / Storage Nodes
        │
        ▼
   Grafana Alloy
        │
        ▼
 Central Loki (10.100.10.18:3100)
        │
        ▼
     Grafana UI
````

---

# Step 1 — Install Grafana Alloy (Each Node)

Run this on every node (PVE, PBS, storage):

```bash
curl -L -O https://github.com/grafana/alloy/releases/latest/download/alloy-linux-amd64.zip
unzip alloy-linux-amd64.zip
sudo mv alloy-linux-amd64 /usr/local/bin/alloy
sudo chmod +x /usr/local/bin/alloy
```

---

# Step 2 — Create Configuration Directory

```bash
sudo mkdir -p /etc/alloy
```

---

# Step 3 — Create Alloy Configuration

Create the config file:

```bash
sudo nano /etc/alloy/config.alloy
```

---

## Base Configuration Template

Replace `host` and `vlan` per node.

```hcl
loki.source.journal "system" {
  forward_to = [loki.write.default.receiver]

  labels = {
    host = "CHANGE_ME"
    vlan = "CHANGE_ME"
  }
}

loki.write "default" {
  endpoint {
    url = "http://10.100.10.18:3100/loki/api/v1/push"
  }
}
```

---

# Step 4 — Node Label Mapping

Use **names (not VLAN IDs)** for readability.

| Node Type            | host        | vlan    |
| -------------------- | ----------- | ------- |
| Proxmox VE           | pve-node-1  | mgmt    |
| Proxmox Backup (PBS) | pbs-backups | backup  |
| Storage Server       | pve-storage | storage |

---

## Example: PBS Node (VLAN 50)

```hcl
labels = {
  host = "pbs-backups"
  vlan = "backup"
}
```

---

## Example: PVE Node (VLAN 99)

```hcl
labels = {
  host = "pve-node-1"
  vlan = "mgmt"
}
```

---

## Example: Storage Node (VLAN 60)

```hcl
labels = {
  host = "pve-storage"
  vlan = "storage"
}
```

---

# Step 5 — Create systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/alloy.service
```

---

## Service Configuration

```ini
[Unit]
Description=Grafana Alloy
After=network-online.target

[Service]
ExecStart=/usr/local/bin/alloy run /etc/alloy/config.alloy
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

# Step 6 — Enable and Start Alloy

```bash
sudo systemctl daemon-reload
sudo systemctl enable alloy
sudo systemctl start alloy
```

---

# Step 7 — Verify Alloy is Running

Check service status:

```bash
systemctl status alloy
```

Follow logs:

```bash
journalctl -u alloy -f
```

---

# Step 8 — Verify Logs in Grafana

In Grafana:

1. Go to **Explore**
2. Select **Loki**
3. Run queries:

### All logs

```logql
{}
```

### Filter by host

```logql
{host="pbs-backups"}
```

### Filter by VLAN group

```logql
{vlan="backup"}
```

---

# Result

Once configured:

* Every node sends logs to central Loki
* Logs are labeled by:

  * `host`
  * `vlan`
* No additional agents required per service
* VLANs only affect routing, not logging logic

---

# Key Design Principle

> VLANs handle network segmentation.
> Labels handle log organization.

They are independent systems.

---

# Optional Next Improvements

If you want to extend this setup later:

* Add file log collection (`/var/log/auth.log`, syslog)
* Add Docker container log shipping
* Add TLS/auth between Alloy and Loki
* Auto-detect hostname instead of manual config
* Add log filtering (reduce noise)

