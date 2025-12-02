# VPN Comparison: Tailscale vs. MikroTik RouterOS VPN

This document compares **Tailscale** and **MikroTik's built-in VPN options** for remote access and networking in a home lab environment. It highlights the strengths, weaknesses, and use cases for each solution.

---

## Overview

| Feature | Tailscale | MikroTik RouterOS VPN |
|---------|-----------|---------------------|
| **VPN Type** | Peer-to-peer mesh VPN (WireGuard-based) | Traditional VPN (IPSec, L2TP, OpenVPN, WireGuard, EoIP, etc.) |
| **Connectivity Scope** | Node-to-node; can optionally advertise LAN subnets | Central router gateway; provides access to entire LAN/subnets |
| **Setup Complexity** | Very simple; install client & authenticate | Moderate to complex; requires VPN server setup, keys, firewall/NAT configuration |
| **NAT Traversal** | Automatic | Requires port forwarding or public IP for remote clients |
| **Scalability** | Excellent; mesh network grows easily | Moderate; central VPN server can become bottleneck with many clients |
| **Device Compatibility** | Cross-platform (Linux, Windows, Mac, mobile, containers) | Dependent on supported VPN protocols; legacy devices may require custom config |
| **Performance** | Direct peer-to-peer tunnels → lower latency | All traffic routes through central VPN → potential backflow/double-hop delays |
| **Access Control** | Identity-based ACLs, per-device permissions | Firewall rules and routing define access; manual configuration |
| **Dependency** | Relies on Tailscale control plane (for peer discovery & NAT fallback) | Fully self-hosted; no external dependency |
| **Use Case Best Suited** | Easy remote access, mixed devices, dynamic environments, cloud/home-lab mesh VPN | Site-to-site tunnels, full LAN access from remote, legacy devices, advanced routing/firewall setups |

---

## Advantages

### Tailscale
- Simple to deploy: install, authenticate, done.
- Peer-to-peer connections minimize latency.
- Automatic NAT traversal; works behind CG-NAT or firewalls.
- Scales easily as more nodes are added.
- Identity-based ACLs for granular access control.
- Cross-platform: works on laptops, servers, mobile devices, and containers.

### MikroTik VPN
- Full control over routing, firewall, VLANs, and subnet access.
- Supports multiple VPN protocols for compatibility with various devices.
- Self-hosted; no reliance on external services.
- Ideal for site-to-site VPNs or accessing all LAN devices from a single tunnel.
- Good for complex or enterprise-style network topologies.

---

## Disadvantages

### Tailscale
- Each node must run Tailscale to be directly accessible.
- Limited control over advanced routing/firewall rules.
- Reliance on Tailscale control plane for peer discovery and NAT fallback.
- Non-Tailscale devices require a subnet router to be reachable.

### MikroTik VPN
- More complex setup: keys, firewall rules, port forwarding, routing.
- NAT traversal behind CG-NAT requires extra configuration.
- Centralized tunnel can create a performance bottleneck.
- Adding new clients requires manual configuration.

---

## Recommended Use Cases

| Scenario | Recommended Solution |
|----------|--------------------|
| Access individual home lab nodes and services quickly and easily | Tailscale |
| Connect multiple remote clients to entire LAN, including legacy devices | MikroTik VPN |
| Mixed OS environment with laptops, servers, and mobile devices | Tailscale |
| Site-to-site VPN between multiple physical networks | MikroTik VPN |
| Want minimal setup and low maintenance | Tailscale |
| Full control over routing, firewall, and complex network topologies | MikroTik VPN |

---

## Practical Recommendation for Home Labs

- **If each node is capable of running Tailscale (e.g., Wyse nodes, servers):**
  - Install Tailscale on each node for direct, low-latency remote access.
  - Use the router purely for routing, firewall, and internet connectivity.
- **If you need full LAN access, or have devices that cannot run Tailscale:**
  - Consider using MikroTik VPN to provide a central VPN gateway.
- Combining both approaches is possible:
  - Use Tailscale on nodes for convenience and performance.
  - Keep MikroTik VPN for site-to-site tunnels or legacy device access.

## Using a Single Tailscale Node as a Subnet Router

If you want remote access to devices on your LAN that **cannot run Tailscale**, you can configure a single node (e.g., a Wyse 5070 home lab node) as a **subnet router**. This allows Tailscale to provide VPN access to your entire LAN through one node.

### How It Works

1. Install Tailscale on a single gateway node  
   - Choose a powerful, always-on node (e.g., Wyse 5070 with 32GB RAM).  
   - Install and authenticate the Tailscale client.

2. Enable subnet routing  
   - Configure Tailscale on the node to advertise the LAN subnet. For example, if your LAN is 192.168.88.0/24:  
   sudo tailscale up --advertise-routes=192.168.88.0/24  
   - Approve the advertised route in the Tailscale admin console.

3. Configure MikroTik static routes  
   - On your MikroTik router, add a static route pointing to the Tailscale node as the gateway for the advertised subnet. For example:  
   /ip route add dst-address=192.168.88.0/24 gateway=<Tailscale_Node_IP>

#### Traffic Flow Through Subnet Router

Remote Client → Tailscale Network → Subnet Router Node (Wyse 5070) → LAN Devices (non-Tailscale nodes)

**Note:** This example may result in data backflow, where traffic from remote clients to certain LAN devices passes through the subnet router node. This setup is intended for illustration; in production, plan routing carefully to minimize latency and avoid unnecessary hops.

4. Result  
   - Remote devices connected to your Tailscale tailnet can now access all devices in your LAN, including those that do not run Tailscale.  
   - The gateway node handles routing and NAT for non-Tailscale devices.  
   - Other Tailscale nodes or devices do not require additional setup.


### Advantages
   - Provides remote access to the full LAN without installing Tailscale on every device.
   - Centralizes subnet routing on a single, capable node.
   - Keeps the router free from VPN overhead.

### Considerations
   - The subnet router node becomes a single point of entry — it must remain powered and connected.
   - Some traffic may take an extra hop through the subnet router, potentially adding slight latency compared to direct Tailscale connections.
   - Proper firewall rules on the gateway node are recommended to secure LAN access.
   - Traffic from remote clients to LAN devices that do not run Tailscale will flow through the subnet router node, potentially adding extra hops and latency. The example provided is illustrative—plan routing carefully in production to minimize backflow and ensure optimal performance.



## References / Further Reading
- [Tailscale Documentation](https://tailscale.com/kb/) – Official Tailscale guides, setup instructions, and advanced features.  
- [MikroTik VPN Documentation](https://wiki.mikrotik.com/wiki/Manual:IP/IPsec) – Official MikroTik documentation covering VPN protocols, configuration, and best practices.  
- [Tailscale Subnet Router Guide](https://tailscale.com/kb/1100/subnets/) – How to configure a node to act as a subnet router for your LAN.  
- [MikroTik RouterOS VPN Overview](https://wiki.mikrotik.com/wiki/Manual:Interface/VPN) – Overview of VPN types and configuration in RouterOS.

---

*End of Document*
