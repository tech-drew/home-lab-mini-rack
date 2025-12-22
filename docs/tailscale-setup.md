# Setting Up Tailscale VPN for a Linux Home Lab

This guide explains how to set up **Tailscale** to securely connect a Fedora laptop to debian-based home lab nodes without configuring port forwarding or modifying your ISP modem.

Tailscale is built on **WireGuard** but handles NAT traversal automatically, making it ideal for networks where direct VPN traffic is blocked.

> **Note:** Some ISPs block WireGuard VPN traffic at the modem/router level. Tailscale bypasses this restriction by encapsulating traffic and managing NAT traversal, allowing secure connectivity to your home lab devices without modifying the ISP modem.

---

## Prerequisites

* Fedora laptop (client) — commands shown are for Fedora/RHEL-based systems
* Debian-based home lab nodes (servers) — commands shown are for Ubuntu/Kubuntu/Debian
* Admin/root access on all nodes
* Internet connectivity
* Tailscale account (free tier is sufficient)
* Commands may vary slightly for other distributions

---

## 1. Install Tailscale

### On Fedora (Laptop)

```bash
sudo dnf install tailscale
sudo systemctl enable --now tailscaled
```

### On Debian-based (Lab Nodes)

```bash
sudo apt update
sudo apt install -y tailscale
sudo systemctl enable --now tailscaled
```

---

## 2. Connect Devices to Tailscale

You can connect devices in two ways:

### Option A: Headless / SSH Setup Using an Auth Key

1. On another device with a browser, go to:
   [https://login.tailscale.com/admin/authkeys](https://login.tailscale.com/admin/authkeys)
2. Generate a **single-use** or **reusable auth key**.
3. On the debian-based node:

```bash
sudo tailscale up --authkey <YOUR_AUTH_KEY> --hostname your-node-hostname
```

* Replace `<YOUR_AUTH_KEY>` with your tailscale authorization key.
* Replace `your-node-hostname` with a descriptive name for this node.
* This method is ideal for headless devices without a browser.

### Option B: Manual CLI Login

If you prefer not to use an auth key:

```bash
sudo tailscale up
```

* A URL will be displayed in the terminal:

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

* You should see all connected devices with their Tailscale IPs (`100.x.x.x`).
* Test connectivity:

```bash
ping 100.x.x.x   # Replace with a node's Tailscale IP
ssh user@100.x.x.x
```

---

## 4. Access Services

Use Tailscale IPs to access services on your home lab nodes:

```bash
ssh user@100.x.x.x
curl http://100.x.x.x:8080
```

No port forwarding or firewall changes are required on your home network.

---

## 5. Optional: Subnet Routing

To access an entire home lab subnet (e.g., `192.168.88.x`) via Tailscale:

1. Pick a debian-based node to act as a **subnet router**.
2. Advertise routes on that node:

```bash
sudo tailscale up --advertise-routes=192.168.88.0/24
```

3. Enable the route in the **Tailscale admin console**.

> This allows your laptop to access all nodes via LAN IPs instead of Tailscale IPs.

---

## 6. Additional Tips

* Tailscale handles **NAT traversal**, bypassing ISP modem restrictions.
* Firewall rules on your home lab nodes may need adjustments if they are restrictive.
* Tailscale works **peer-to-peer**, encrypting VPN traffic end-to-end.
* This setup is **Linux-friendly**, ideal for Fedora and debian-based environments.
* Assign descriptive hostnames to easily identify nodes in `tailscale status`.

---

