# Setting Up WireGuard VPN on MikroTik and Connecting from Fedora Linux

This guide explains how to replace the default MikroTik **QuickSet L2TP/IPsec VPN** with a modern, faster, and simpler **WireGuard VPN** — and how to connect to it from Fedora Linux.  
WireGuard is easier to configure, more stable, and works natively on Linux.

## Why Use QuickSet L2TP/IPsec VPN?

The MikroTik **QuickSet L2TP/IPsec VPN** is designed for simple, plug-and-play remote access and may still be useful in certain situations:

- **Cross-platform compatibility** — works natively on Windows, macOS, iOS, and Android without additional software.  
- **No need for kernel modules** — clients do not require WireGuard, Tailscale, or other third-party tools.  
- **Legacy support** — useful if connecting to older devices or networks that do not support WireGuard or Tailscale.  
- **Centralized configuration** — QuickSet automates many IPsec settings, making it easier for less technical users to deploy a VPN quickly.

However, for modern setups, especially Linux-focused environments or when using **Tailscale**, **WireGuard-based VPNs are generally faster, simpler to maintain, and more reliable**. Tailscale builds on WireGuard, providing automatic key management, NAT traversal, and secure networking without the configuration overhead of L2TP/IPsec.

---

## Prerequisites

Before setting up the WireGuard VPN, make sure you have the following:

- **MikroTik RouterOS 7 or higher** — WireGuard is supported only on RouterOS v7+  
- **Admin access to your MikroTik router** — required to create interfaces, peers, and firewall rules  
- **Fedora Linux client** — tested on Fedora 42+, but WireGuard works on other Linux distros as well  
- **Basic knowledge of networking** — understanding of IP addresses, subnets, and NAT  
- **Public IP or DDNS hostname** — needed for the VPN Endpoint on the client configuration  
- **Firewall access** — ensure the WireGuard port (default: 13231) is open to allow client connections  
- **Terminal access on Fedora** — for installing WireGuard tools and creating configuration files  

> **Note:** Some ISPs lock down consumer modems (e.g., Arris Surfboard S33) and do **not allow port forwarding or bridge mode**. This may prevent external clients from reaching your MikroTik WireGuard server. You may need to contact your ISP or use a bridge-capable modem for proper VPN access.

---

# 1. Disable or Ignore QuickSet VPN
The default QuickSet “VPN Access” uses L2TP/IPsec. You do not need to delete it — just stop using it.  
WireGuard will be configured separately under **Interfaces**.

---

# 2. Create a WireGuard Interface on MikroTik

1. Go to **Interfaces → WireGuard → Add New**  
2. Configure:
   - **Name:** `wg-vpn`
   - **Listen Port:** `13231` (or any port you prefer)
   - Leave **Private Key** auto-generated
3. Click **Apply**, then **OK**  

Your **Public Key** will appear after saving — you will need it for Fedora.

---

# 3. Assign the WireGuard Interface an IP

MikroTik WebFig / WinBox:  
Go to **IP → Addresses → Add New**, then fill in:

- **Address:** `10.10.10.1/24`  
- **Interface:** `wg-vpn`  

Click OK. This makes MikroTik the VPN gateway for your clients.

---

# 4. Create a WireGuard Peer for Fedora

1. Go to **Interfaces → WireGuard**, select `wg-vpn`  
2. Go to **Peers → Add New**  
3. Fill in:
   - **Public Key:** (from Fedora client — generated later)  
   - **Allowed Address:** `10.10.10.2/32`  
   - **Comment:** `fedora-laptop`  

Do not enter endpoint info for the client; it is only needed on the server side for remote peers.

---

# 5. Add NAT Rule for VPN Clients (Optional)

If you want VPN clients to access the internet:

1. Go to **IP → Firewall → NAT → Add**  
2. Set:
   - **Chain:** `srcnat`
   - **Out Interface:** your WAN interface (example: `ether1`)  
   - **Src Address:** `10.10.10.0/24`  
   - **Action:** `masquerade`  

---

# 6. Configure Firewall Rules

- Input chain: allow UDP port `13231` from WAN  
- Forward chain: allow traffic between `wg-vpn` and your LAN (bridge)  
- **Note:** Your firewall rules may vary depending on your network topology. You may need to troubleshoot connectivity if pings or handshakes fail.

---

# 7. Install WireGuard on Fedora

```bash
sudo dnf install wireguard-tools
````

---

# 8. Generate Client Keys on Fedora

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

This creates:

* `privatekey` — Fedora client’s private key
* `publickey` — public key to paste into MikroTik peer configuration

---

# 9. Create WireGuard Client Configuration on Fedora

Create `/etc/wireguard/wg-mikrotik.conf`:

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.10.10.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <MIKROTIK_PUBLIC_KEY>
AllowedIPs = 192.168.88.0/24
Endpoint = vpn.example.com:13231
PersistentKeepalive = 25
```

* Replace `<CLIENT_PRIVATE_KEY>` and `<MIKROTIK_PUBLIC_KEY>` with actual keys.
* Replace `vpn.example.com` with your public IP or DDNS domain.

> **Tip:** `AllowedIPs = 192.168.88.0/24` routes only LAN traffic through the VPN. Using `0.0.0.0/0` will route all internet traffic via VPN, which is usually not desired.

---

# 10. Bring Up WireGuard on Fedora

```bash
sudo wg-quick up wg-mikrotik
sudo wg-quick down wg-mikrotik
sudo systemctl enable wg-quick@wg-mikrotik
```

* Verify handshake and traffic with `sudo wg`
* Test ping to MikroTik LAN IP (`10.10.10.1`) and LAN devices

> **Note:** If you cannot connect:
>
> * Ensure the WAN modem/router forwards UDP port `13231` (or is in bridge mode)
> * Check MikroTik firewall rules for input/forward chains
> * NAT may be needed if routing internet traffic

---

# 11. Using NetworkManager Applet (Optional)

On Fedora, after installing the `NetworkManager-wireguard` plugin:

* Open network settings → add a new WireGuard connection
* Enter client private key, MikroTik public key, endpoint, and allowed IPs
* Enable auto-connect for graphical management

This provides the same functionality as `wg-quick` with a GUI.

---

# Notes on Modem / ISP Restrictions

* Consumer modems from some ISPs (like Arris Surfboard S33) may **block incoming UDP traffic**.
* If you are connecting from outside your LAN, the WireGuard handshake may **never reach MikroTik**.
* Recommended solution:

  1. Put the modem in **bridge mode** so your MikroTik receives the public IP
  2. If bridge mode is not possible, request **port forwarding** for the WireGuard port
  3. Alternatively, use a VPS relay or reverse tunnel to bypass modem restrictions

Once the modem allows the UDP port through, both Fedora and Windows clients should be able to connect without additional configuration changes.


