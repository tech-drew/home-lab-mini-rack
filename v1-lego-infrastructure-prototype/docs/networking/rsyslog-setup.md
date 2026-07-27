# Homelab Syslog Server Setup on Kubuntu

This guide explains how to set up a centralized **syslog server** on a Kubuntu VM to collect logs from your Proxmox cluster and your router.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install Rsyslog](#step-1-install-rsyslog)
- [Step 2: Configure Rsyslog to Receive Remote Logs](#step-2-configure-rsyslog-to-receive-remote-logs)
- [Step 3: Open Firewall Ports](#step-3-open-firewall-ports)
- [Step 4: Restart Rsyslog](#step-4-restart-rsyslog)
- [Step 5: Configure Proxmox Nodes](#step-5-configure-proxmox-nodes)
- [Step 6: Configure Router](#step-6-configure-router)
- [Step 7: Organize Logs by Host](#step-7-organize-logs-by-host)
- [Optional: Enhanced Logging](#optional-enhanced-logging)

---

## Prerequisites

- Kubuntu VM with sudo privileges
- Proxmox nodes
- Router with syslog support
- Basic networking configured (VM reachable by nodes and router)

---

## Step 1: Install Rsyslog

```bash
sudo apt update
sudo apt install rsyslog
````

---

## Step 2: Configure Rsyslog to Receive Remote Logs

Edit the configuration:

```bash
sudo nano /etc/rsyslog.conf
```

Enable UDP and TCP syslog reception by adding/uncommenting:

```conf
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")
```

Save and exit.

---

## Step 3: Open Firewall Ports

If UFW is enabled:

```bash
sudo ufw allow 514/udp
sudo ufw allow 514/tcp
sudo ufw reload
```

---

## Step 4: Restart Rsyslog

```bash
sudo systemctl restart rsyslog
sudo systemctl enable rsyslog
sudo systemctl status rsyslog
```

---

## Step 5: Configure Proxmox Nodes

On each Proxmox node, edit `/etc/rsyslog.conf` or create `/etc/rsyslog.d/remote.conf`:

```conf
*.* @<KUBUNTU_VM_IP>          # UDP
*.* @@<KUBUNTU_VM_IP>         # TCP
```

Restart rsyslog on the Proxmox node:

```bash
sudo systemctl restart rsyslog
```

---

## Step 6: Configure Router

Most routers allow sending logs to a remote syslog server:

* **Syslog server IP:** `<KUBUNTU_VM_IP>`
* **Port:** `514`
* **Protocol:** UDP (recommended)

Apply and save settings.

---

## Step 7: Organize Logs by Host

Create a dedicated log file for each host:

```bash
sudo nano /etc/rsyslog.d/remote.conf
```

Add:

```conf
$template RemoteLogs,"/var/log/%HOSTNAME%/syslog.log"
*.* ?RemoteLogs
& stop
```

Create log directories:

```bash
sudo mkdir /var/log/<hostname>
sudo chown syslog:adm /var/log/<hostname>
```

Restart rsyslog:

```bash
sudo systemctl restart rsyslog
```

Logs will now be sorted by hostname.

---

## Optional: Enhanced Logging

For a more advanced setup, you can:

* Enable **log rotation** via `/etc/logrotate.d/`
* Install **`logwatch`** or **`Graylog`/`ELK stack`** for searchable logs and dashboards
* Secure remote logging over **TLS** (recommended for production)

---

## References

* [Rsyslog Documentation](https://www.rsyslog.com/doc/)
* [Proxmox Syslog Configuration](https://pve.proxmox.com/wiki/System_logs)
* [Router Syslog Setup Guides](https://wiki.dd-wrt.com/wiki/index.php/Syslog)

---

> By following this guide, you’ll have a centralized logging server collecting all your Proxmox and router logs in one place, ready for monitoring and troubleshooting.

