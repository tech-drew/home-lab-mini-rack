# Home Lab: Mini-Rack

The primary objective of this home lab is to design and implement an environment that closely replicates enterprise IT infrastructure in both architecture and operational practices.

In addition to this core objective, the project emphasizes several secondary priorities: minimizing physical footprint, optimizing energy efficiency, reducing noise levels, and maintaining strict cost control. 

To meet these tight design constraints and add a touch of creative engineering, the physical infrastructure is housed inside a custom-built, open-concept enclosure constructed out of Lego and Mega Bloks.

---

<img width="4000" height="2252" alt="20260622_082125" src="https://github.com/user-attachments/assets/c2eb1acb-a849-453e-be73-43161214c35c" />

<img width="2713" height="2250" alt="20260622_085314" src="https://github.com/user-attachments/assets/5091c224-cf18-4e43-9bda-60eb9c379409" />


# The Physical Chassis: Block-Rack

Rather than utilizing a traditional, expensive, and heavy consumer metal rack, the core infrastructure (including the thin clients, modem, and router) is housed in a custom-engineered enclosure built entirely out of Lego and Mega Bloks. 

As a structural building material, plastic blocks possess two distinct engineering weaknesses: **Weight limits** and **Heat tolerance (ABS plastic degradation)**. 

The architecture of this lab explicitly designs around these constraints:

* **Low Structural Weight:** The Dell Wyse 5070 nodes are ultra-lightweight thin clients. Stacking them does not exert enough downward force to stress the interlocking clutch power of the Lego and Mega Bloks chassis.
* **Passive & Low-Heat Design:** The compute nodes utilize energy-efficient, low-wattage architectures that output minimal thermal energy. Coupled with an open-air vertical rack design and supplementary USB cooling fans, internal ambient heat remains well below the thresholds required to compromise the integrity of the plastic blocks.

This custom block-rack allows the entire enterprise stack to scale dynamically, operate silently, and use a minimal physical footprint.

---

# Project Status: Work in Progress

This repository documents the planning and design of my personal home lab project. It is currently a work in progress and includes design notes, hardware plans, and intended setup steps.

Updates will be added as the lab is built, configured, and iterated on. Some documentation may become out of date over time.

---

# Hardware Components

**Note:** This home lab is being assembled for approximately **$665 total out-of-pocket** by scavenging parts and repurposing hardware already owned or acquired inexpensively.

---

## 1. Modem

* **Model:** Arris Surfboard S33
* **Cost:** Already owned (~$130 typical retail)

---

## 2. Router

* **Model:** MikroTik RB5009
* **Cost:** Already owned (~$260 typical retail)
* **Note:** Extensive research went into selecting this device. The reasoning is documented in the *Networking Equipment Analysis* section.

---

## 3. Access Points

* **Model:** MikroTik cAP ax
* **Cost:** Already owned (~$120 each typical retail)
* **Note:** See *Networking Equipment Analysis* for design reasoning.

---

## 4. Proxmox Virtual Environment Compute Nodes

* **Model:** Dell Wyse 5070 Thin Client (×4)
* **Cost:** $155.98 total (used, purchased 11/28/2025 via eBay)

### Upgrades

* 32 GB DDR4 RAM (reused from previous projects)
* 256 GB M.2 SATA SSDs (see Storage section)

### Storage Compatibility

* NVMe SSDs are **not supported**
* Requires M.2 SATA (2260 or 2280)

### Power Draw (per node)

* Idle: ~5–6 W
* Light load: 7–10 W
* Full load: 13–15 W

### Notes

* Updating to the latest BIOS is recommended.
* These systems are reliable for 24/7 operation.
* The Intel J5005 memory controller effectively limits usable RAM to ~30 GB.

### Memory Behavior

**Linux:** Installing 32 GB results in ~30 GB usable due to hardware addressing limits. This does not affect stability.

**Windows:** To utilize full installed memory reporting:

* Disable Secure Boot
* Add boot option:

```bash
bcdedit /set {current} truncatememory 0x800000000

```

The limitation is due to the CPU memory controller and cannot be resolved via configuration changes.

---

## 5. Storage for Wyse Clients

* **Model:** SK Hynix 256 GB M.2 2280 SATA SSD (×4)
* **Cost:** $73.63 total (used, purchased 12/13/2025 via eBay)

### Notes

* Fully compatible with Wyse 5070 SATA M.2 slot
* Used SSDs were avoided due to wear concerns
* Adequate for Proxmox, Docker, and lightweight Kubernetes workloads
* SMART monitoring will be used for health tracking

---

## 6. Proxmox Storage Node & Backup Server

* **Model:** HP EliteDesk 800 G4 SFF (×2)
* **Cost:** $303.36 total (used, purchased 02/06/2026 via eBay)

---

### CPU Configuration (included in total cost calculation)

| System Role | CPU | Cost |
| --- | --- | --- |
| Storage / NAS Node | i7-8700T (6C/12T, 35W) | $86.68 |
| Backup Server | i5-8500T (6C/6T, 35W) | $45.39 |

**CPU subtotal:** $132.07

---

### Storage Configuration

* Boot drives: 256 GB SATA SSD (×2 total) — Already owned
* Data drives: 2 TB SATA SSD (×4 total) — Already owned
* Configured in mirrored pairs for redundancy

**Note:** SSD prices are expensive right now. The 2TB drives I used are not necessary for this project. My backup server is currently using about 100GB of storage and my storage server is using about 50GB of storage. 256GB SSDs are probaby enough space to get you started on a project like this.

---

### Power Draw (per system)

* Idle: ~18–25 W
* Light load: 30–45 W
* Full load: 65–90 W

---

## 7. UPS

* **Model:** APC BN1500M2
* **Cost:** Already owned
* **Note:** A pure sine wave UPS is a potential future upgrade.

---

# Total Cost of Purchased Materials

| Item | Cost (USD) |
| --- | --- |
| HP EliteDesk 800 G4 SFF Systems | $303.36 |
| Intel Core i5-8500T CPU | $45.39 |
| Intel Core i7-8700T CPU | $86.68 |
| Dell Wyse 5070 Thin Clients (×4) | $155.98 |
| 256 GB M.2 SATA SSDs (×4) | $73.63 |
| **Total** | **$665.04** |

---

### Pricing Disclaimer

Due to fluctuating hardware markets, pricing shown reflects historical purchase conditions and may not represent current market values. Many components were sourced from secondary markets or acquired prior to recent tech pricing volatility.

---

# Virtualization & Containers

* Proxmox VE for virtualization and container management
* Virtual machines (VMs) and LXC containers managed through Proxmox VE, with some VMs running Docker-based application stacks

---

# Self-Hosted Services

* Backup automation and snapshotting
* Monitoring and logging stacks
* Services such as Uptime Kuma and Grafana Alloy + Loki

---

# Networking Experiments

* VLAN segmentation using MikroTik RB5009
* Firewall rules, VPNs, and routing labs
* Multi-tier network simulation environments

---

# Trade-offs and Limitations

This lab prioritizes learning, efficiency, and cost control over raw compute performance.

* Thin clients provide limited CPU performance but extremely low power consumption
* Storage capacity is intentionally conservative
* Enterprise-grade redundancy is deferred due to cost constraints
* Incremental upgrades are preferred over large upfront investments

---

# Design Philosophy

This project emphasizes learning through iteration and experimentation. Systems are expected to evolve through cycles of building, testing, failure, refinement, and improvement.

Significant financial investment is intentionally deferred until real-world requirements are better understood. Version 1 of this lab relies on repurposed and cost-efficient hardware sourced from previous projects and secondary markets.

Future revisions may include:

* Upgrading networking infrastructure beyond 1 GbE (minimum 2.5 GbE, with 10+ GbE preferred for storage-heavy workloads)
* Increasing PCIe expansion availability for networking and storage flexibility
* Evaluating higher-performance systems as workload demands increase

These improvements will be balanced against power consumption, heat output, and total operating cost.
