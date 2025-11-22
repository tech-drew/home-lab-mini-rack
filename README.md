# Mini-Rack Home Lab

This project documents the creation of a compact home lab setup, designed for **self-hosting resources** and **studying to become a Cloud Engineer**. The goal is to have a fully functional 8U mini-rack that supports networking, compute nodes, and power management in a compact form factor.

---

## Hardware Components

### 1. Rack
- **Model:** DeskPi Rack T1 (8U)  
- **Price:** $150  
- **Link:** [Amazon](https://www.amazon.com/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0CSCWVTQ7/ref=sr_1_2?crid=GDHRRH2UXRHZ&dib=eyJ2IjoiMSJ9.ctEHtcyHjljLot4TDmYNl004xVLEQxLpZ6ugneAQoWVkhRg059_UPw-724bbB1bXFfm6LohuAbtqAsw3Y_ZEye7Qc4MZkfSZiiaLHag1DHMgBd1no0EKbJ0-mZXOYH0hgPWFCIV2uGIPh01S6aZSlOyJRZ_7x74oMpAZ0z-fGocnI-kJR2dEKfzSLEl4ZJqIBLE0dZ3dHUGQY5rIRj6Tt9sq7-mFjNTY1nl513uWYpI.pNpAxwKHSyFE8YLJR-L5Uv-rvytRnUTUx0zTvjoTYb8&dib_tag=se&keywords=GeeekPi%2B8U%2BServer%2BCabinet&qid=1763829355&sprefix=geeekpi%2B8u%2Bserver%2Bcabinet%2Caps%2C230&sr=8-2&th=1)

### 2. Modem
- **Model:** Arris Surfboard S33 (already owned)  
- **Rackmount Kit Price:** $50  
- **Link:** [Etsy Rackmount Kit](https://www.etsy.com/listing/1417205328/arris-surfboard-s33-or-s34-cable-modem?ls=s&ga_order=most_relevant&ga_search_type=all&ga_view_type=gallery&ga_search_query=10in+1u+rack+mount+harris+surfboard+s33&ref=sr_gallery-1-1&frs=1&sts=1&nob=1&content_source=a8a40f39-286e-4662-affb-a93538c956e1%253ALT4639b8691b02d74bea7c089b26a2100b70423dc1&organic_search_click=1&logging_key=a8a40f39-286e-4662-affb-a93538c956e1%3ALT4639b8691b02d74bea7c089b26a2100b70423dc1)

### 3. Router
- **Model:** Mikrotik RB5009 (already owned)  
- **Rackmount Kit Price:** $20  
- **Link:** [Amazon Rackmount Kit](https://www.amazon.com/Mikrotik-K-79-rackmount-RB5009-L009/dp/B0CP9RDHC9/ref=sr_1_1?crid=3JPU92J043ZPW&dib=eyJ2IjoiMSJ9.kqfbn0GGT0hLOoMfehekaCxKF6iYwuAKecbf8s8E5x1JrbW5WXkBcDwgCT6fw6Hed4Moqo1IiLo1w0d4IT4t14Hs_54_im-Wv4d6fjXitt9H92vLC-9U-FL0rbQOR1fCTGoJMj4gds2V3Qgym_hRn8wOz0Xxhwdz4QWSC6IT5mJqCaJ739fC8ODE6Qkm6JX4pxzGMeZwvNDzNu5bc6I9tXB-1LflI1-PBLunENUxtW8.tAFV2LMuHuGIoo_RTHrNG9Ckk6NCpr10dZTHvTrRMFM&dib_tag=se&keywords=mikrotik+rb5009+rackmount+kit&qid=1763833281&sprefix=mikrotik+rb5009%2Caps%2C185&sr=8-1)

### 4. Compute Nodes
- **Model:** Dell Wyse 5070 Thin Client × 4  
- **Price:** ~$160 total (used / eBay)  
- **Link:** [eBay Wyse 5070](https://www.ebay.com/sch/i.html?_nkw=wyse+5070+thin+client&_sacat=0&_from=R40&_trksid=p4624852.m570.l1311)  
- **Upgrades:** Each node will be equipped with **32GB DDR4 RAM** and **SSDs** from previous projects for improved performance.

### 5. Power Management
- **PDU:** 1U rackmount PDU installed at the bottom of the rack for clean power distribution.  
- **UPS:** Located externally due to the shallow depth of the mini-rack.

---

## **Rack Layout**
```bash
U1: (Top) Modem - Arris Surfboard S33
U2: Router - Mikrotik RB5009
U3: Patch Panel
U4: Dell Wyse 5070 Thin Client #1
U5: Dell Wyse 5070 Thin Client #2
U6: Dell Wyse 5070 Thin Client #3
U7: Dell Wyse 5070 Thin Client #4
U8 (Bottom): PDU
```

**Notes on Layout:**

- Networking equipment is at the top for easy monitoring and cable management.  
- Compute nodes occupy the middle 4U for stability and airflow.  
- PDU at the bottom keeps the center of gravity low and allows clean upward routing of power cables.  
- UPS is external due to mini-rack depth constraints.

---

## **Project Goals**

- Create a **compact, low-power, yet powerful home lab** for experimentation and learning.  
- Support **self-hosted applications**, **containerized workloads**, and **distributed systems labs**.  
- Simulate **multi-node environments** typical in cloud engineering, with redundancy and HA setups.  

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
