# Home Lab: Mini-rack

This project documents the creation of a compact home lab setup, designed for **self-hosting resources** and **studying to become a Cloud Engineer**. The goal is to have a fully functional 8U mini-rack that supports networking, compute nodes, and power management in a compact form factor.

---

## Project Status: Work in Progress

This repo documents the planning and design of my personal home-lab project. 
Currently, this is a work-in-progress with design notes, hardware plans, and intended setup steps.

Updates will be added as the lab is built and configured.

---

## Hardware Components
**Notes:** I’m putting this homelab together for about $229 by scavenging parts and jury-rigging whatever I have or can find to make it all work.

### 1. Modem
- **Model:** Arris Surfboard S33 (already owned)  


### 2. Router
- **Model:** Mikrotik RB5009 (already owned)  
- **Note:** Extensive research went into deciding on the networking equipment for this project. The reasoning and analysis behind choosing the MikroTik RB5009 and cAP ax can be found in the [Networking Equipment Analysis](docs/networking-equipment-analysis.md) document.


### 3. Access Points
- **Model:** MikroTik cAP ax (already owned)
- **Price:** ~$120 each (typical retail)
- **Link:** [MikroTik cAP ax](https://mikrotik.com/product/cap_ax)
- **Note:** Extensive research went into deciding on the networking equipment for this project. The reasoning and analysis behind choosing the MikroTik RB5009 and cAP ax can be found in the [Networking Equipment Analysis](docs/networking-equipment-analysis.md) document.

### 4. Compute Nodes
- **Model:** Dell Wyse 5070 Thin Client × 4  
- **Price:** ~$155 total (used / Purchased 11/28/2025 on eBay)  
- **Link:** [eBay Wyse 5070](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client&_sacat=0&_from=R40&_trksid=p4624852.m570.l1311)  
- **Upgrades:** Each node will be equipped with **32GB DDR4 RAM** from previous projects and **512GB 2280 sata SSDs** will be purchased for improved performance.
- These clients do not support nvme 2280 SSDs. m.2 sata 2260/2280 SSDs must be used. 
- **Power Draw:**
- Idle: ~5–6 W
- Light load: 7–10 W
- Full CPU load: 13–15 W

### Notes:
- Updating to the latest BIOS on these clients is recommended for stability.
- These clients are reported to be reliable when running 24/7.
- Memory: MemTest86 shows the J5005 CPU can efficiently address only ~30 GB of RAM; operations beyond that are slower but stable.
- **Linux:** Using 32 GB of RAM will result in 30 GB usable; the remaining 2 GB are unaddressed due to CPU memory controller limitations, but this does not negatively impact system stability.
- **Windows:** To use 32 GB of RAM, Secure Boot must be disabled in BIOS and the following boot switch added:
- `bcdedit /set {current} truncatememory 0x800000000`
- After this, Windows will report 32 GB (~29.8 GB usable).
- The memory limitation is due to the CPU memory controller; rank configuration (single vs. dual) does not affect this.
- With 32 GB of RAM and Secure Boot enabled, Windows will not boot; the boot switch only works with Secure Boot disabled.

### 5. Storage
- **Model:** SK Hynix 256 GB M.2 2280 SATA SSD  
- **Price:** ~$74 total (used / Purchased 12/13/2025 on Ebay)  
- **Link:** [SK Hynix 256 GB M.2 2280 SATA SSD ](https://www.ebay.com/itm/388921986094)  
- **Notes:**  
  - Ideal for Wyse 5070 thin clients (SATA-only M.2 2280 slot).  
  - Used SSDs are cheaper ($30–35) but have some wear. The cost savings don''t justify the risk of issues at this price point.  
  - Sufficient for Linux, Proxmox, Docker, or lightweight Kubernetes workloads.  
  - Ensure BIOS is updated and monitor health with SMART tools periodically.
  - I don't have a preference for this specific SSD brand. I choose whatever what cheapest with good reviews for this project.
  
### 6. UPS (already owned)
- **Model:** APC BN1500M2
- **Notes:** I’ll upgrade to a pure-sine-wave UPS when I can. For now, I’m improvising with the equipment I have.

### Total Cost of Materials

| Item                 | Cost (USD) |
|---------------------|------------|
| Computer Nodes      | $155       |
| Storage (SSDs 4 × 256 GB)   | $74       |
| **Total**           | **$229**   |


---

## Rack v1 layout

| U Position   | Device                          |
|--------------|---------------------------------|
| U1 (Top)     | Modem – Arris Surfboard S33     |
| U2           | Router – Mikrotik RB5009        |
| U3           | Dell Wyse 5070 Thin Client #1   |
| U4           | Dell Wyse 5070 Thin Client #2   |
| U5           | Dell Wyse 5070 Thin Client #3   |
| U6           | Dell Wyse 5070 Thin Client #4   |
| U7           | Power Bricks                    |
| U8 (Bottom)  | Power Bricks                    |

**Notes on Layout:**

- Networking equipment is at the top for easy monitoring and cable management.  
- Compute nodes occupy rack space towards the middle of the rack for stability and airflow.
- As of this time the setup does not include a Patch Panel, fans or PDUs. These items will be added in a future v2 build of the rack.
- UPS is external due to mini-rack space constraints.

---

## **Project Goals**

- Create a **compact, low-power, efficient home lab** for experimentation and learning.  
- Support **self-hosted applications**, **containerized workloads**, and **distributed systems labs**.  
- Simulate **multi-node environments** typical in cloud engineering, with redundancy and HA setups.  

---

## Trade-offs and Limitations

This home lab setup has been carefully designed for **learning, experimentation, and self-hosting**, with a focus on **cost efficiency** and **compactness**. Some trade-offs were made due to budget constraints and hardware limitations:

1. **Compute Capacity**
   - Dell Wyse 5070 thin clients provide modest CPU performance (Intel J5005) and limited memory addressing (~30 GB usable per 32 GB upgrade).  
   - These nodes are **passively cooled and use very little power** (5–15 W depending on load), which is a significant benefit for a small, always-on lab.  
   - High-performance nodes with more CPU power and memory typically cost **$200–$400 each**, which is outside the current budget.

2. **Scalability**
   - With 4 compute nodes, the lab is ideal for **small multi-node experiments**.  
   - Expanding the number of nodes is possible, but additional nodes will require more rack space, power, and cooling considerations.

3. **Power Management**
   - Included PDUs are basic 4-outlet mini rack PDUs.  
   - Managed PDUs with **SNMP and SSH capabilities** provide better monitoring and control but cost around **$300 each**, which is beyond the current budget.  
   - Power and cable management is sufficient for this setup, but upgrading to managed PDUs is a potential future improvement.

4. **Storage**
   - M.2 SATA SSDs (256 GB) provide basic storage for OS and lightweight workloads.  
   - 512 GB SATA SSDs would be optimal. However, the 256 GB SSDs were $17 each where as the 512 GB ssds were $60 each. I am trying to keep cost low so I bought SSDs bases on the best value.
   - I don't know what I don't know. I did not want to spend a ton of money on hardware until I have experiemented with this home lab and learn what I really need. Hence the SSD space compromise.
   - Storage can be expanded later if larger workloads are needed.

5. **Overall Design**
   - The lab prioritizes **learning and experimentation** over raw performance.  
   - It provides a **fully rack-mounted, low-power, and compact solution** that supports cloud engineering studies, self-hosted applications, and multi-node environments.

**Conclusion:** This setup represents a balanced compromise between **budget, functionality, and expandability**, with the added benefit of low-power, passively cooled compute nodes. It’s an ideal starting point for students or hobbyists learning cloud computing, networking, and self-hosted infrastructure without incurring high costs.


---

## **Future Plans**

- Explore virtualization platforms like **Proxmox**, **Kubernetes**, or **Docker Swarm**.  
- Add monitoring, logging, and self-hosted services such as **Nextcloud**, **Home Assistant**, or **personal CI/CD tools**.  
- Experiment with networking setups using **Mikrotik RB5009 features** like VLANs, firewalls, and VPNs.  

---

## **References / Links**

- [Dell Wyse 5070 Thin Client](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client)  

---

*This project is open-source and intended for educational purposes and home lab experimentation.*
