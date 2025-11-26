# Mini-Rack Home Lab

This project documents the creation of a compact home lab setup, designed for **self-hosting resources** and **studying to become a Cloud Engineer**. The goal is to have a fully functional 8U mini-rack that supports networking, compute nodes, and power management in a compact form factor.

---

## Project Status: Work in Progress

This repo documents the planning and design of my personal home-lab project. 
Currently, this is a work-in-progress with design notes, hardware plans, and intended setup steps.

Updates will be added as the lab is built and configured.

---

## Hardware Components
**Notes:** I’m putting this homelab together for about $600 by scavenging parts and jury-rigging whatever I have or can find to make it all work.
### Rack
- **Model:** DeskPi Rack T1 (8U)  
- **Price:** $150  
- **Link:** [Amazon](https://www.amazon.com/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0CSCWVTQ7/ref=sr_1_2?crid=GDHRRH2UXRHZ&dib=eyJ2IjoiMSJ9.ctEHtcyHjljLot4TDmYNl004xVLEQxLpZ6ugneAQoWVkhRg059_UPw-724bbB1bXFfm6LohuAbtqAsw3Y_ZEye7Qc4MZkfSZiiaLHag1DHMgBd1no0EKbJ0-mZXOYH0hgPWFCIV2uGIPh01S6aZSlOyJRZ_7x74oMpAZ0z-fGocnI-kJR2dEKfzSLEl4ZJqIBLE0dZ3dHUGQY5rIRj6Tt9sq7-mFjNTY1nl513uWYpI.pNpAxwKHSyFE8YLJR-L5Uv-rvytRnUTUx0zTvjoTYb8&dib_tag=se&keywords=GeeekPi%2B8U%2BServer%2BCabinet&qid=1763829355&sprefix=geeekpi%2B8u%2Bserver%2Bcabinet%2Caps%2C230&sr=8-2&th=1)

<img width="1400" height="788" alt="image" src="https://github.com/user-attachments/assets/42d80b2b-fe66-42c3-a23b-09caf1b81e0e" />
_An example of the DeskPI Rack T1 (8U) rack._


### 1. Modem
- **Model:** Arris Surfboard S33 (already owned)  
- **Rackmount Kit Price:** $50  
- **Link:** [Etsy Rackmount Kit](https://www.etsy.com/listing/1417205328/arris-surfboard-s33-or-s34-cable-modem?ls=s&ga_order=most_relevant&ga_search_type=all&ga_view_type=gallery&ga_search_query=10in+1u+rack+mount+harris+surfboard+s33&ref=sr_gallery-1-1&frs=1&sts=1&nob=1&content_source=a8a40f39-286e-4662-affb-a93538c956e1%253ALT4639b8691b02d74bea7c089b26a2100b70423dc1&organic_search_click=1&logging_key=a8a40f39-286e-4662-affb-a93538c956e1%3ALT4639b8691b02d74bea7c089b26a2100b70423dc1)

### 2. Router
- **Model:** Mikrotik RB5009 (already owned)  
- **Rackmount Kit Price:** $20  
- **Link:** [Amazon Rackmount Kit](https://www.amazon.com/Mikrotik-K-79-rackmount-RB5009-L009/dp/B0CP9RDHC9/ref=sr_1_1?crid=3JPU92J043ZPW&dib=eyJ2IjoiMSJ9.kqfbn0GGT0hLOoMfehekaCxKF6iYwuAKecbf8s8E5x1JrbW5WXkBcDwgCT6fw6Hed4Moqo1IiLo1w0d4IT4t14Hs_54_im-Wv4d6fjXitt9H92vLC-9U-FL0rbQOR1fCTGoJMj4gds2V3Qgym_hRn8wOz0Xxhwdz4QWSC6IT5mJqCaJ739fC8ODE6Qkm6JX4pxzGMeZwvNDzNu5bc6I9tXB-1LflI1-PBLunENUxtW8.tAFV2LMuHuGIoo_RTHrNG9Ckk6NCpr10dZTHvTrRMFM&dib_tag=se&keywords=mikrotik+rb5009+rackmount+kit&qid=1763833281&sprefix=mikrotik+rb5009%2Caps%2C185&sr=8-1)

### 3. Patch Panel
- **Model:** Keystone 12 Port Pass Thru Patch Panel 
- **Price:** ~$30 total (new / Amazon)  
- **Link:** [Patch Panel](https://www.amazon.com/Keystone-Support-Rapink-Shielded-Removable/dp/B09MTH3V14?pd_rd_w=HYv4P&content-id=amzn1.sym.53b72ea0-a439-4b9d-9319-7c2ee5c88973&pf_rd_p=53b72ea0-a439-4b9d-9319-7c2ee5c88973&pf_rd_r=HZYWPMP2SHM60SM6AWF1&pd_rd_wg=mejGb&pd_rd_r=fb67cfb8-fd95-449d-93e8-f6d026ee34aa&pd_rd_i=B09MTH3V14&psc=1&linkCode=sl1&tag=mmjjg-20&linkId=ed0e9e391744888147c556d160c23bee&language=en_US&ref_=as_li_ss_tl)  

### 4. Compute Nodes
- **Model:** Dell Wyse 5070 Thin Client × 4  
- **Price:** ~$160 total (used / eBay)  
- **Link:** [eBay Wyse 5070](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client&_sacat=0&_from=R40&_trksid=p4624852.m570.l1311)  
- **Upgrades:** Each node will be equipped with **32GB DDR4 RAM** and **256GB 2280 SSDs** from previous projects for improved performance.
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

### 5. SSD Upgrades
- **Model:** KingSpec 512 GB M.2 2280 SATA SSD  
- **Price:** ~$40 new / $30–35 used  
- **Link:** [Amazon KingSpec 512GB SATA SSD](https://www.amazon.com/KingSpec-512GB-Internal-Solid-NT-512/dp/B078MJHLCM/ref=sr_1_8?crid=209PTSVCAW3IL&dib=eyJ2IjoiMSJ9.UGh2mgu-tw2epCkB-ZBVrTGg1GDeeAUSLqYfQkuR4wCe8RLb1cb-oScjsBgWaYd6oSOoFiX4uZ-JzWNL0tD-WemctgN7dBqw4LFXytBB_VBpFhyVQtJfDsmqj0kKBGza3O_nV3MtFJYvMiF0aGfn2hmYCMHZa64BfVzU67VCUUgEarp4VqaZ3z9MIU-Y-6T_Q89HdvrCs2HCCJ0UmdiVGPZBtSPR14ohWZU8klWHSRk.yWz2mwjt4ReNXf6Wq6Lqa6d1mSQXNeZebsts8hVcoDA&dib_tag=se&keywords=m.2%2B2280%2Bsata%2Bssd%2B512gb&qid=1764137917&sprefix=m.2%2Bsata%2Bssd%2B2280%2B512%2Caps%2C179&sr=8-8&xpid=sNDlYSr9GFtA2&th=1)  
- **Notes:**  
  - Ideal for Wyse 5070 thin clients (SATA-only M.2 2280 slot).  
  - Used SSDs are cheaper ($30–35) but have some wear. The cost savings don''t justify the risk of issues at this price point.  
  - Sufficient for Linux, Proxmox, Docker, or lightweight Kubernetes workloads.  
  - Ensure BIOS is updated and monitor health with SMART tools periodically.
  
### 6. PDU
- **Model:** TP1713 4-outlet Mini Rack PDU x2
- **Price:** ~$120 total (new / Amazon)  
- **Link:** [10 in rack PDU](https://www.amazon.com/ElecVoztile-Protection-Overload-RackMount-Cabinets/dp/B0FF41T167/ref=sr_1_1?crid=1WL8UQSKZXZYV&dib=eyJ2IjoiMSJ9.Bu0xSY1SswhayzQUYafiCVnMgWqZf7fvv3qKdg9d1AGpgUOmJEdXOIIr7ylUTGq7i3NnL9WZVRqvBorF7xhuCvQYFD_0EH-lZNpRMneySlcYcYLzhzFLUF15mRy8xv4kGzYUPWArDgqGqPt-NDaBFfMdt1inNSlKZdHbLbP5OtJliK33masBTWjkzrZILBM5hlHAniGZna8IlfoQ0dwXGWgSye7bBpfkFqMYMoehc8zScQj30N4j7yfauxFMdDsFLGQJSD99bSlhgntMJvB24VXoQggodXchdYr053393OM.Y9Qg3B--ZmarkMDFWcW3CA0f5BKpmq-XpSVypOsti8A&dib_tag=se&keywords=TP1713+4-outlet+Mini+Rack+PDU&qid=1763837647&s=electronics&sprefix=tp1713+4-outlet+mini+rack+pdu%2Celectronics%2C182&sr=1-1)
- **Notes:** There are better small form factor PDUs that support snmp and ssh for monitoring. The PDUs with these capabilities are around $300 each. I need two small form factor PDUs so it would be $600 just for PDUs. $600 for PDUs is outside the budget for this project.

### 6. UPS (already owned)
- **Model:** APC BN1500M2
- **Notes:** I’ll upgrade to a pure-sine-wave UPS when I can. For now, I’m improvising with the equipment I have.

### Total Cost of Materials

| Item                 | Cost (USD) |
|---------------------|------------|
| Rack                | $150       |
| Modem Rack Mount    | $50        |
| Router Rack Mount   | $20        |
| Patch Panel         | $30        |
| Computer Nodes      | $160       |
| SSDs (4 × 512 GB)   | $160       |
| PDU                 | $120       |
| Misc Patch Cables   | $30        |
| **Total**           | **$720**   |


---

## Rack Layout

| U Position   | Device                          |
|--------------|---------------------------------|
| U1 (Top)     | Modem – Arris Surfboard S33     |
| U2           | Router – Mikrotik RB5009        |
| U3           | Patch Panel                     |
| U4           | Dell Wyse 5070 Thin Client #1   |
| U5           | Dell Wyse 5070 Thin Client #2   |
| U6           | Dell Wyse 5070 Thin Client #3   |
| U7           | Dell Wyse 5070 Thin Client #4   |
| U8 (Bottom)  | PDUs                             |

**Notes on Layout:**

- Networking equipment is at the top for easy monitoring and cable management.  
- Compute nodes occupy rack space towards the middle of the rack for stability and airflow.  
- PDUs at the bottom keeps the center of gravity low and allows clean upward routing of power cables.  
- UPS is external due to mini-rack space constraints.

---

## **Project Goals**

- Create a **compact, low-power, yet powerful home lab** for experimentation and learning.  
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
   - M.2 SATA SSDs (512 GB) provide adequate storage for OS and lightweight workloads.  
   - Used SSDs are cheaper ($30–35), but reliability is higher when purchasing new ($40).  
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

- [DeskPi Rack T1](https://www.amazon.com/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0CSCWVTQ7)  
- [Arris Surfboard S33 Rackmount](https://www.etsy.com/listing/1417205328/arris-surfboard-s33-or-s34-cable-modem)  
- [Mikrotik RB5009 Rackmount Kit](https://www.amazon.com/Mikrotik-K-79-rackmount-RB5009-L009/dp/B0CP9RDHC9)  
- [Dell Wyse 5070 Thin Client](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client)  

---

*This project is open-source and intended for educational purposes and home lab experimentation.*
