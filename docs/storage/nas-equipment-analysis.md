# NAS Equipment Analysis

**Notes:** Research for NAS hardware focused on building **small, low-power, reliable, and flexible** storage systems suitable for a home lab environment. These systems are intended to run **Proxmox VE or TrueNAS** installed directly on bare metal.

A key requirement is support for **booting from SATA or NVMe SSDs**, as neither Proxmox nor TrueNAS operate well when installed on **eMMC-only storage**. Flexibility and long-term reliability are prioritized over turnkey or proprietary NAS solutions.

Networking is currently limited to **1 GbE** due to constraints of the existing router and Proxmox nodes. While **10 GbE or 10 Gb SFP+** networking is preferred—particularly for a High Availability NAS—it is not a hard requirement at this stage. Poor networking performance is not a deal breaker given current infrastructure, but better networking is desirable for future expansion.

---

## NAS Platform Comparison

| Platform                         | CPU / Power Profile        | Storage Support | Networking                           | Pros                                       | Cons                                                     | Notes / Suitability                                                            |
| -------------------------------- | -------------------------- | --------------- | ------------------------------------ | ------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Aoostar WTR Pro (8-core)**     | Low-power x86 (~15 W TDP)  | SATA + NVMe     | 1 GbE / 2.5 GbE (revision dependent) | Compact form factor                        | Reported controller and reliability issues               | Hardware reliability concerns make it unsuitable for long-term NAS use.        |
| **Beelink ME Mini Pro**          | Low-power mobile x86       | NVMe only       | 2.5 GbE                              | Extremely compact, quiet                   | Soldered RAM, limited expandability, higher storage cost | Size-efficient but inflexible. NVMe-only design increases overall system cost. |
| **GMKtec NucBox G9 Pro**         | Mobile x86                 | NVMe            | 2.5 GbE                              | Small footprint                            | Documented overheating and throttling issues             | Thermal instability makes it a poor choice for 24/7 NAS workloads.             |
| **Minisforum MS01**              | Hybrid P-core / E-core CPU | NVMe + SATA     | 10 GbE                               | Strong I/O potential, excellent networking | Expensive, OS scheduling issues on some platforms        | Powerful but likely overkill for a low-power NAS.                              |
| **HP EliteDesk G4 (SFF / Mini)** | Desktop-class x86          | SATA + NVMe     | 1 GbE (PCIe expandable)              | Flexible, reliable, PCIe expansion         | Larger physical size, higher idle power                  | Strong long-term option; PCIe allows future 10 GbE upgrade.                    |

---

## High Availability NAS Considerations

For a **High Availability NAS**, **random read and write performance** is important. Many HA and shared-storage workloads involve frequent small I/O operations rather than large sequential transfers. Because of this, **SSD-based storage pools** are strongly preferred over HDD-only configurations.

Key characteristics for a High Availability NAS include:

* SSD-backed storage pools to support low-latency random I/O
* Reliable 24/7 operation
* Sufficient CPU and memory for ZFS or similar filesystems
* Networking that can scale to 10 GbE when the rest of the lab supports it

While high-speed networking improves performance, the benefits are currently limited by the rest of the environment. Storage performance and system reliability remain the primary concerns.

---

## Backup NAS Considerations

A **Backup NAS** has very different requirements. Backup workloads are dominated by **large, sequential writes and reads**, and are generally not sensitive to random I/O latency.

For this role:

* **HDD-based storage pools** are acceptable and cost-effective
* SSDs are still recommended for boot drives
* 1 GbE networking is sufficient for typical backup workloads
* Emphasis is placed on capacity, reliability, and cost per terabyte

High random I/O performance is not critical for a backup-focused system.

---

## Summary

* **High Availability NAS:** Benefits significantly from SSD storage pools due to random read/write workloads. Networking improvements are helpful but secondary to storage performance and reliability.
* **Backup NAS:** Focuses on capacity and reliability. HDD-based storage is appropriate, and performance requirements are modest.
* **Consumer mini-PC NAS platforms** often trade reliability and expandability for size, which can be problematic for long-term NAS use.
* **Business-class systems** provide better flexibility, OS compatibility, and upgrade paths.

---

## Conclusion

For this home lab, flexibility and reliability are prioritized over extreme compactness or peak performance. While modern mini-PC NAS platforms offer appealing specifications, many suffer from limitations such as thermal instability, soldered components, or storage constraints.

Older business-class systems like the **HP EliteDesk G4** offer a better balance. Although physically larger and limited to 1 GbE out of the box, they provide proven reliability, broad OS compatibility, and a clear upgrade path through PCIe expansion.

This makes them well-suited for both backup and high availability roles as the lab evolves.

---

## Key Takeaways

* Avoid **eMMC-only systems** for Proxmox or TrueNAS.
* **SSD storage pools matter** for High Availability NAS workloads.
* **HDDs are sufficient** for backup-focused NAS systems.
* Thermal reliability and expandability are more important than compact size.
* PCIe expansion provides long-term flexibility.

