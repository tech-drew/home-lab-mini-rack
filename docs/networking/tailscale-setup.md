# Setting Up Tailscale VPN for a Linux Home Lab

This guide explains how to set up **Tailscale** to securely connect a Fedora laptop (client) to your Debian-based home lab nodes, including a Proxmox cluster, without configuring port forwarding or modifying your ISP modem.

Tailscale is built on **WireGuard**, but it handles NAT traversal automatically. This makes it ideal for networks where direct VPN traffic may be blocked.

> **Note:** Some ISPs block WireGuard VPN traffic at the modem/router level. Tailscale bypasses this by encapsulating traffic and managing NAT traversal, allowing secure connectivity to your home lab devices without modifying your ISP modem.

---

## Prerequisites

* Fedora laptop (client) — commands are shown for Fedora/RHEL-based systems
* Debian-based home lab nodes (servers, including Proxmox nodes)
* Admin/root access on all nodes
* Internet connectivity
* Tailscale account (free tier is sufficient)
* Commands may vary slightly for other Linux distributions

---

## 1. Install Tailscale

### On Fedora (Laptop)

```bash
sudo dnf install tailscale
sudo systemctl enable --now tailscaled
````

### On Debian-based Lab Nodes (Including Proxmox Hosts)

```bash
sudo apt update
sudo apt install -y tailscale
sudo systemctl enable --now tailscaled
```

---

## 2. Connect Devices to Tailscale

You can connect devices in two ways:

### Option A: Headless / SSH Setup Using an Auth Key

1. On another device with a browser, go to [Tailscale Auth Keys](https://login.tailscale.com/admin/authkeys).
2. Generate a **single-use** or **reusable auth key**.
3. On the node:

```bash
sudo tailscale up --authkey <YOUR_AUTH_KEY> --hostname <DESCRIPTIVE_NODE_NAME>
```

* Replace `<YOUR_AUTH_KEY>` with your Tailscale authorization key.
* Replace `<DESCRIPTIVE_NODE_NAME>` with a clear hostname (e.g., `node1`, `prox-node1`).

> Ideal for headless devices without a browser.

---

### Option B: Manual CLI Login

If you prefer not to use an auth key:

```bash
sudo tailscale up
```

* A URL will appear in the terminal:

```
To authenticate, visit: https://login.tailscale.com/a/XXXXXXX
```

* Open this URL on any device with a browser, log in to your Tailscale account, and approve the device.
* The node will automatically join your Tailscale network after approval.

---

## 3. Verify Connectivity

```bash
tailscale status
tailscale ip
```

* All connected devices should appear with their Tailscale IPs (`100.x.x.x`).
* Test connectivity between devices:

```bash
ping 100.x.x.x   # Replace with another node's Tailscale IP
ssh user@100.x.x.x
```

---

## 4. Access Services

You can now access services on your home lab nodes using Tailscale IPs:

```bash
ssh user@100.x.x.x
curl http://100.x.x.x:8080
```

No port forwarding or firewall changes are required on your home network.

---

## 5. Optional: Subnet Routing

To access your entire home lab subnet (e.g., `192.168.88.x`) via Tailscale:

1. Pick a Debian-based node (or Proxmox host) to act as a **subnet router**.
2. Advertise routes on that node:

```bash
sudo tailscale up --advertise-routes=192.168.88.0/24
```

3. Enable the advertised route in the **Tailscale admin console**.

> This allows your laptop or other devices to access nodes via LAN IPs instead of Tailscale IPs.

---

## 6. Run Tailscale as a Service (Optional)

Tailscale usually installs a systemd service automatically. Ensure it runs on boot:

```bash
sudo systemctl enable --now tailscaled
sudo systemctl status tailscaled
```

---

## 7. Notes and Best Practices

* **Do not run `tailscale up --advertise-routes` on the master Proxmox node** unless you intend for it to route your LAN traffic.
* Use **Tailscale IPs** for cluster management, backups, or even accessing the Proxmox web interface remotely.
* Keep `/etc/hosts` entries for internal networking; Proxmox clusters rely on these hostnames.
* Assign descriptive hostnames for clarity in `tailscale status`.
* Tailscale handles **NAT traversal** automatically, bypassing most ISP restrictions.
* Firewall rules on home lab nodes may need adjustments if restrictive.

---

## 8. Proxmox Cluster Considerations

* Each Proxmox node must have **outbound internet access**.
* Tailscale installation on Proxmox is the same as Debian-based servers.
* Once installed, nodes will appear in your Tailscale network and can communicate securely without exposing ports to the internet.

---

This setup allows you to securely manage and access your home lab from anywhere, without complex VPN configuration or port forwarding.
