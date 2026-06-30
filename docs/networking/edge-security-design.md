# Edge Security Design

This repository documents the Edge-Security Design changes I would consider implementing if I decide to add a Intrustion Prevention System (IPS) and Intrution Detection System (IDS) into the existing environmnet. 

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

