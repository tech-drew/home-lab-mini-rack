# Edge Security Design

This repository documents the advanced architecture built on top of the secure VLAN design described in the primary network design document.

Rather than replacing the original design, this document extends it by introducing a **Routed LAN with a Transit Network (No-NAT)** architecture that combines high-performance local routing on the MikroTik RB5009 with advanced edge security provided by Firewalla.

The overall goal remains consistent with the rest of this homelab: to create a **reasonable mirror of enterprise networking practices** while remaining practical for a home environment.

---

# Network Topology Overview

```text
       [ ISP Modem ]
             |
             | Public IP (DHCP)
             v
 [ Firewalla Gold/Purple ]
 Edge Firewall
 NAT • IDS • IPS • Scheduling
             |
      10.0.0.0/30 Transit
             |
             v
 [ MikroTik RB5009 ]
 Core L3 Router
 VLAN Gateway
 CAPsMAN Controller
             |
      VLAN Infrastructure
             |
   +---------------------------+
   | VLAN 10  SERVERS          |
   | VLAN 20  TRUSTED          |
   | VLAN 30  GUEST            |
   | VLAN 40  IOT              |
   | VLAN 50  BACKUP           |
   | VLAN 60  STORAGE          |
   | VLAN 99  MGMT             |
   +---------------------------+
             |
      Management VLAN
             |
      Tailscale Node
```

---

# Architectural Goals

This design preserves the principles established in the primary network architecture while adding modern edge security capabilities.

Objectives include:

* Enterprise-inspired separation of core routing and edge security
* High-performance local routing and storage networking
* Preservation of the existing VLAN architecture
* Per-device visibility at the network edge
* Secure remote administration
* Minimal disruption to the existing homelab

---

# Design Constraints

Several design decisions from the primary architecture are intentionally preserved.

## Core Routing Remains on the RB5009

The RB5009 continues to provide:

* Inter-VLAN routing
* VLAN gateway services
* CAPsMAN controller functions
* Local switching
* Jumbo Frame support for the Storage network

Keeping these functions local ensures storage replication, VM migration, and other east-west traffic remain fast and independent of the internet edge.

---

## Existing Wireless Architecture

The wireless design remains unchanged.

CAPsMAN continues operating as a Layer-2 extension of the wired VLAN architecture.

Wireless clients inherit the same VLAN membership and firewall policies as wired clients.

As documented in the primary design, MikroTik Multi-Passphrase (MPSK) provides VLAN assignment in place of enterprise RADIUS authentication, which is intentionally omitted from this home lab due to consumer device compatibility.

---

## Mixed MTU Architecture

The mixed-MTU strategy from the primary design remains unchanged.

Infrastructure-only networks continue using MTU 9000:

* VLAN 50 (BACKUP)
* VLAN 60 (STORAGE)

All client VLANs continue using MTU 1500 for maximum compatibility while allowing storage replication and Proxmox migration traffic to benefit from Jumbo Frames.

---

# Routed LAN with a Transit Network (No NAT)

Instead of using Double NAT, the Firewalla functions solely as the internet edge while the RB5009 remains the core router.

## Separation of Responsibilities

### Firewalla

Responsible for:

* Public NAT
* Internet edge security
* IDS/IPS
* Internet policy enforcement
* Device scheduling
* WAN firewalling

Firewalla provides inline IDS/IPS and policy enforcement suitable for an advanced home and small-office environment.

### MikroTik RB5009

Responsible for:

* Inter-VLAN routing
* VLAN gateways
* CAPsMAN
* Local switching
* Infrastructure management
* Storage networking

This separation allows each platform to perform the tasks for which it is best suited.

---

# Why Not Double NAT?

A traditional Double NAT deployment would have been simpler to configure but introduces several operational limitations.

| Double NAT                               | Routed LAN                                |
| ---------------------------------------- | ----------------------------------------- |
| Simple initial setup                     | Slightly more routing configuration       |
| Limited visibility of downstream devices | Per-device visibility at the edge         |
| Two independent firewall domains         | Single edge security policy               |
| More difficult troubleshooting           | Clear routing boundaries                  |
| Better suited for consumer deployments   | Better reflects enterprise network design |

Although Routed LAN requires static routing, it provides a cleaner and more scalable architecture while preserving high-performance local routing.

---

# Disabling NAT on the RB5009

The default masquerade rule is removed from the RB5009.

As a result, devices retain their original private IP addresses when communicating through the Firewalla.

Instead of observing all outbound traffic as originating from a single router address, Firewalla can identify each individual device across every VLAN.

This improves:

* Device visibility
* Security monitoring
* Policy assignment
* Event logging
* Incident investigation

---

# Transit Network

A dedicated `/30` transit network connects the Firewalla to the RB5009.

Example:

```text
Firewalla LAN : 10.0.0.1/30
RB5009 WAN    : 10.0.0.2/30
```

Static routes on the Firewalla direct traffic destined for each internal VLAN toward the RB5009.

This preserves the existing VLAN architecture while allowing Firewalla to remain the single internet gateway.

### Operational Consideration

Each routed VLAN requires a corresponding static route on the Firewalla.

As additional VLANs are introduced, their associated routes should be added during deployment to maintain end-to-end connectivity.

---

# Security Philosophy

This homelab follows a LAN-first security model. Internal services are expected to remain fully functional regardless of internet availability. Internet connectivity is treated as an optional service rather than an operational requirement. This minimizes dependence on external networks while reducing opportunities for compromised devices to communicate outside the trusted environment.

---

# Internet Access Policy (Zero Trust-Inspired)

This architecture adopts a **default-deny internet access policy** that treats the public internet as an inherently hostile environment.

Rather than assuming devices should always have unrestricted internet connectivity, internet access is permitted only when operationally necessary.

This approach aligns with Zero Trust principles by minimizing unnecessary external communication while preserving normal local network operation.

---

# Scheduled Internet Lockdown

Firewalla enforces scheduled outbound internet access policies for the entire network.

During predefined periods—such as overnight while the household is asleep or during normal work hours—all outbound internet access is blocked for every production VLAN.

This includes:

* VLAN 10 (SERVERS)
* VLAN 20 (TRUSTED)
* VLAN 30 (GUEST)
* VLAN 40 (IOT)
* VLAN 50 (BACKUP)
* VLAN 60 (STORAGE)

Local communication—including file sharing, backups, storage replication, virtualization management, and other inter-VLAN services—continues to operate according to RB5009 routing and firewall policies.

Only communication destined for the public internet is restricted.

significantly reduces the network's external attack surface from:

* Contacting command-and-control infrastructure
* Downloading additional malware
* Participating in botnets
* Exfiltrating data
* Initiating unauthorized outbound connections

Because the RB5009 continues handling all local routing, storage replication, VM migration, backups, and other east-west traffic remain fully operational during scheduled internet isolation.

---

# Management Network Internet Policy

The Management VLAN (VLAN 99) follows an even stricter security model.

Unlike the other VLANs, VLAN 99 does **not** have unrestricted internet access during normal operation.

Instead, internet access is limited to only the traffic required for secure remote administration via Tailscale.

Tailscale is treated as a trusted management overlay rather than general internet access.

All other outbound internet traffic originating from VLAN 99 is denied by default.

General internet access from the Management VLAN is enabled only temporarily when administrative maintenance requires downloading updates, such as:

* RouterOS upgrades
* Proxmox updates
* TrueNAS updates
* Linux package updates
* Firmware downloads

Once maintenance is complete, unrestricted internet access is disabled again, returning VLAN 99 to its default-deny state.

This minimizes the exposure of critical infrastructure systems while still allowing routine maintenance when explicitly authorized.

---

# Secure Remote Management with Tailscale

A dedicated Tailscale node resides within VLAN 99 (MGMT).

Consistent with the primary network design:

* Infrastructure management remains isolated within VLAN 99.
* Guest and IoT devices cannot administer infrastructure.
* Management systems use static addressing.
* No inbound firewall ports are exposed to the public internet.

The only outbound internet communication permanently permitted from VLAN 99 is that required to maintain the encrypted Tailscale control connection.

Because Tailscale establishes outbound encrypted sessions using stateful connections, remote administrative access remains available even while all other internet access is blocked.

---

## Why Remote Access Continues to Work

During scheduled internet isolation:

```text
Internet
    |
Blocked by Firewalla
    |
+-----------------------------+
| VLAN 10  BLOCKED            |
| VLAN 20  BLOCKED            |
| VLAN 30  BLOCKED            |
| VLAN 40  BLOCKED            |
| VLAN 50  BLOCKED            |
| VLAN 60  BLOCKED            |
| VLAN 99  Tailscale ONLY     |
+-----------------------------+
```

When a remote administrator connects through Tailscale:

```text
Remote Administrator
        |
Encrypted Tailscale Tunnel
        |
Dedicated VLAN 99 Tailscale Node
        |
RB5009 Inter-VLAN Routing
        |
Destination Device
```

The Tailscale node maintains its encrypted outbound connection while all other internet access remains prohibited.

Once traffic enters the Management VLAN, the RB5009 routes it internally according to its normal inter-VLAN routing and firewall policies. Local administrative access therefore remains available without exposing management services directly to the public internet.

---

# Security Benefits

This internet access policy provides several advantages:

* Default-deny internet access for all production networks during scheduled isolation periods.
* Infrastructure systems remain inaccessible from the internet except through authenticated Tailscale connections.
* Critical management systems maintain no routine outbound internet connectivity beyond what is required for secure remote administration.
* Local services continue operating normally even while internet access is unavailable.
* Compromised devices have significantly fewer opportunities to communicate with external command-and-control infrastructure or exfiltrate data.

By treating the public internet as an untrusted network and granting internet connectivity only when operationally necessary, the design reinforces the homelab's enterprise-inspired security model while remaining practical to operate and maintain.
