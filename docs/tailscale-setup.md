# Setting Up Tailscale VPN for a Linux Home Lab

This guide explains how to set up **Tailscale** to securely connect a Fedora laptop to a Kubuntu-based home lab without configuring port forwarding or changing your ISP modem.

Tailscale is built on **WireGuard**, but it handles NAT traversal automatically, making it perfect for networks where direct VPN traffic is blocked.

**Note:** Some ISPs block WireGuard VPN traffic at the modem/router level. Using Tailscale allows you to bypass this restriction because it handles NAT traversal automatically and encapsulates traffic, making it possible to connect your home lab devices without modifying the ISP modem.

---

## **Prerequisites**

* Fedora laptop (client)
* Kubuntu home lab nodes (servers)
* Admin/root access on all nodes
* Internet connectivity
* Tailscale account (free tier is sufficient)

---

## **1. Install Tailscale**

### **On Fedora (Laptop)**

```bash
sudo dnf install tailscale
sudo systemctl enable --now tailscaled
```

### **On Kubuntu (Lab Nodes)**

```bash
sudo apt update
sudo apt install tailscale
sudo systemctl enable --now tailscaled
```

---

## **2. Authenticate Devices**

On each device, run:

```bash
sudo tailscale up
```

* A browser URL will appear — open it and log in to your Tailscale account.
* Repeat for all lab nodes.
* Each device will receive a **Tailscale IP** (`100.x.x.x`).

---

## **3. Verify Connectivity**

On the Fedora laptop:

```bash
tailscale status
```

* You should see all your lab nodes listed with their Tailscale IPs.
* Test connectivity:

```bash
ping 100.x.x.x   # Replace with a node's Tailscale IP
ssh user@100.x.x.x
```

---

## **4. Access Services**

Use Tailscale IPs to reach services on lab nodes:

```bash
ssh user@100.x.x.x
curl http://100.x.x.x:8080
```

No port forwarding or firewall changes are needed on your home network.

---

## **5. Optional: Subnet Routing**

If you want to access the entire home lab subnet (`192.168.88.x`) via Tailscale:

1. Pick a Kubuntu node to act as a **subnet router**.
2. On that node, advertise routes:

```bash
sudo tailscale up --advertise-routes=192.168.88.0/24
```

3. Enable the route in the **Tailscale admin console** for your laptop.

> This allows your laptop to access all nodes via LAN IPs instead of Tailscale IPs.

---

## **6. Notes and Tips**

* Tailscale handles **NAT traversal**, so ISP modem restrictions no longer block connectivity.
* Firewall rules on your home lab nodes may require adjustments if you have restrictive policies.
* Tailscale works **peer-to-peer**, so your VPN traffic is encrypted end-to-end.
* This solution is **Linux-friendly**, ideal for Fedora and Kubuntu environments.

---
