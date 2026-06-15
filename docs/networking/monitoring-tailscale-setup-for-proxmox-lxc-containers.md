# Tailscale Setup for Proxmox LXC Containers

## Goal

Enable direct remote access to:

* Uptime Kuma (CT 101)
* Grafana/Loki (CT 301)

using Tailscale without exposing services to the public Internet.

---

## Environment

* Proxmox VE Cluster
* Unprivileged LXC containers
* Proxmox nodes already connected to Tailscale
* Internal service networks:

  * `10.100.99.0/24`
  * `10.100.10.0/24`

---

## Approach Chosen

Install Tailscale directly inside each application container.

Benefits:

* No subnet router required
* No route advertisements
* No public port forwarding
* Per-service Tailscale identity
* Easy access via MagicDNS

---

## 1. Enable TUN Device in the LXC

On the Proxmox host:

```bash
pct set 101 --dev0 /dev/net/tun
pct set 301 --dev0 /dev/net/tun
```

Restart the containers:

```bash
pct restart 101
pct restart 301
```

Verify:

```bash
pct config 101
pct config 301
```

Expected output:

```ini
dev0: /dev/net/tun
```

---

## 2. Verify TUN Access

Enter the container:

```bash
pct enter 101
```

Check:

```bash
ls -l /dev/net/tun
```

Expected:

```text
/dev/net/tun
```

Repeat for CT 301.

---

## 3. Install Required Packages

Inside each container:

```bash
apt update
apt install -y curl ca-certificates gnupg
```

---

## 4. Install Tailscale

Inside each container:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

---

## 5. Start Tailscale

```bash
systemctl enable --now tailscaled
```

Verify:

```bash
systemctl status tailscaled
```

---

## 6. Join the Tailnet

Run:

```bash
tailscale up
```

Follow the authentication URL displayed in the terminal.

---

## 7. Verify Connectivity

Check assigned Tailscale IP:

```bash
tailscale ip -4
```

Check status:

```bash
tailscale status
```

---

## 8. Enable MagicDNS (Recommended)

In the Tailscale Admin Console:

* Settings
* DNS
* Enable MagicDNS

Containers can then be reached by hostname:

```text
uptime-kuma
grafana-loki
```

or:

```text
uptime-kuma.<tailnet>.ts.net
grafana-loki.<tailnet>.ts.net
```

---

## Example Access URLs

### Grafana

```text
http://grafana-loki:3000
```

### Uptime Kuma

```text
http://uptime-kuma:3001
```

(Verify the actual port configured for Kuma.)

---

## Optional: HTTPS via Tailscale Serve

Expose Grafana:

```bash
tailscale serve 3000
```

Expose Uptime Kuma:

```bash
tailscale serve 3001
```

Check configuration:

```bash
tailscale serve status
```

Benefits:

* Automatic HTTPS
* No reverse proxy
* No certificate management
* Restricted to authenticated Tailnet users

---

## Troubleshooting

Verify TUN device:

```bash
ls -l /dev/net/tun
```

Verify Tailscale service:

```bash
systemctl status tailscaled
```

Verify connectivity:

```bash
tailscale ping <hostname>
```

Show node information:

```bash
tailscale status
```
