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
- **Fedora Linux client** — tested on Fedora 38+, but WireGuard works on other Linux distros as well  
- **Basic knowledge of networking** — understanding of IP addresses, subnets, and NAT  
- **Public IP or DDNS hostname** — needed for the VPN Endpoint on the client configuration  
- **Firewall access** — ensure the WireGuard port (default: 13231) is open to allow client connections  
- **Terminal access on Fedora** — for installing WireGuard tools and creating configuration files

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

# 3. Create a WireGuard Peer for Fedora

1. Go to **Interfaces → WireGuard**
2. Select your interface `wg-vpn`
3. Go to **Peers → Add New**
4. Fill in:
   - **Public Key:** (from Fedora client — generated later)
   - **Allowed Address:**  
     ```
     10.10.10.2/32
     ```
   - **Comment:** `fedora-laptop`

Do not enter endpoint information for the client; it is only needed on the server side for remote peers.

---

# 4. Assign a VPN Subnet to WireGuard

Go to **IP → Addresses → Add** and set:

- **Address:** `10.10.10.1/24`
- **Interface:** `wg-vpn`

This makes the MikroTik act as the VPN gateway.

---

# 5. Add NAT Rule for VPN Clients

If you want the VPN to access the internet, add a NAT rule:

1. Go to **IP → Firewall → NAT → Add**
2. Set:
   - **Chain:** `srcnat`
   - **Out. Interface:** your WAN interface (example: `ether1`)
   - **Src. Address:** `10.10.10.0/24`
   - **Action:** `masquerade`

This allows VPN clients to reach external networks.

---

# 6. Install WireGuard on Fedora

Install WireGuard tools:

```bash
sudo dnf install wireguard-tools
```
# 7. Generate Client Keys on Fedora

Generate a private and public key pair for the Fedora WireGuard client:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
This creates two files:

privatekey — your Fedora client’s private key
publickey — the public key you will paste into MikroTik under WireGuard → Peers → Public Key

# 8. Create WireGuard Client Configuration on Fedora

Create the WireGuard configuration file:
`sudo nano /etc/wireguard/wg-mikrotik.conf`

Paste the following template and modify the placeholders:
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.10.10.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <MIKROTIK_PUBLIC_KEY>
AllowedIPs = 0.0.0.0/0
Endpoint = vpn.example.com:13231
PersistentKeepalive = 25

Replace:
<CLIENT_PRIVATE_KEY> — contents of your privatekey file
<MIKROTIK_PUBLIC_KEY> — MikroTik WireGuard interface public key
vpn.example.com — your public IP or Cloudflare DDNS domain
Save and exit.

# 9. Bring Up the WireGuard VPN on Fedora
Start the VPN:
`sudo wg-quick up wg-mikrotik`

Stop the VPN:
`sudo wg-quick down wg-mikrotik`

Enable the VPN at system startup:
`sudo systemctl enable wg-quick@wg-mikrotik`

If successful, you should see output similar to
latest handshake: 2 seconds ago
transfer: 32 KiB received, 28 KiB sent



