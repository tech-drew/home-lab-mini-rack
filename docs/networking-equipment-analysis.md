# Networking Equipment Analysis

**Notes:** Between 2023 and 2025, I conducted research and purchased networking equipment for different environments: MikroTik devices for my home lab in 2023, and Ubiquiti equipment for my parents’ network in 2025.

I evaluated the following platforms:
- Ubiquiti UniFi
- TP-Link Omada
- Ruckus
- MikroTik routing + Ubiquiti Wi-Fi mix
- MikroTik (final choice)

# Networking Equipment Comparison (~$1000 budget, 2 APs)

| Platform                     | Router | AP(s) | Approx. Cost | Pros                                                                 | Cons                                                             | Notes / Suitability                                 |
|-------------------------------|--------|-------|-------------|----------------------------------------------------------------------|-----------------------------------------------------------------|----------------------------------------------------|
| **Ubiquiti UniFi**           | UniFi Security Gateway / USG | 2× U6 Pro | $900-$1000 | - Centralized controller interface (UniFi Network) <br>- Extensive ecosystem of APs, switches, gateways | - Requires cloud/key setup for some features <br>- Some advanced features may require subscription | Good for easy management and monitoring; aesthetically pleasing APs. Great for a polished home lab. |
| **TP-Link Omada**             | TP-Link ER7206 or similar | 2× EAP660 HD | $700-$950 | - Affordable enterprise-like management (Omada SDN) <br>- Plug-and-play setup for APs | - Cloud-based controller can be confusing <br>- Limited advanced routing features | Strong value for cost-conscious setups. Suitable for basic routing + Wi-Fi deployment. |
| **Ruckus**                    | Ruckus ZoneDirector / SmartZone | 2× R650 | $900-$1200 | - High-performance Wi-Fi APs <br>- SmartMesh and beamforming technology | - Expensive, especially for multiple APs <br>- Controller interface less intuitive | Excellent Wi-Fi performance, but may exceed budget. Often used in professional SMB setups. |
| **MikroTik routing + Ubiquiti APs** | RB5009 PoE | 2× U6 Pro | $950-$1100 | - Flexible routing options (MikroTik) <br>- Ubiquiti APs provide good Wi-Fi coverage | - Mixed ecosystem adds management complexity <br>- Two separate management platforms required | Combines powerful routing with polished Wi-Fi. Can leverage strengths of each platform. |
| **MikroTik (routing & APs)** | RB5009 PoE | 2× cAP ax | $800-$1000 | - Affordable and feature-rich <br>- Advanced routing, firewall, and switching options | - Steeper learning curve <br>- Less polished GUI compared to UniFi/Omada | Offers complete control, budget-friendly. Ideal for hands-on configuration and home lab experimentation. Best choice if you want to learn networking deeply. |

**Summary:**  
- **Budget-friendly & flexible:** MikroTik for routing and APs.  
- **Ease of management:** UniFi or Omada.  
- **Wi-Fi performance focus:** Ruckus, but may exceed $1000 for 2 APs.  
- **Mixed ecosystem:** MikroTik + Ubiquiti if you want a balance of routing power and polished Wi-Fi.  

**Conclusion**

For my setup, I chose MikroTik equipment due to its enterprise-grade features, flexibility, and extensive configuration options, making it ideal for learning advanced networking in a home lab environment. The RB5009 PoE router handles routing, firewall, VLANs, and other Layer 3 functionality for my needs.

One notable limitation is that MikroTik Wi-Fi lacks robust client roaming support. While a single MikroTik AP performs well, multiple APs in the same network often exhibit poor handoff behavior, making roaming unreliable. Based on my experience as a SysAdmin with a CCNA background, roaming was a consistent problem that I was unable to fully resolve. Some users report success using advanced CAPsMAN configurations, but in practice, achieving smooth multi-AP roaming with MikroTik APs can be difficult. I would not recommend using MikroTik APs for multi-AP Wi-Fi deployments. These issues were observed on RouterOS 7.10–7.17.

In hindsight, a more balanced approach for multi-AP setups would be to pair the MikroTik RB5009 router with Ubiquiti APs managed via a Cloud Key.

I have not personally tested TP-Link Omada or Ruckus equipment. I deployed a full Ubiquiti setup at my parents’ house with multiple APs, and Wi-Fi stability and performance were excellent. However, the Ubiquiti Dream Router 7 has a relatively underpowered CPU and is more prosumer-focused than enterprise-grade in terms of features.

For my home network and lab, the MikroTik router alone meets all routing and firewall requirements. While next-generation enterprise routers exist, they are generally cost-prohibitive for home use. If more advanced firewall or routing capabilities are needed in the future, adding an OPNsense router is a viable option.

**Key Takeaways**
- **Single AP setups:** MikroTik APs perform well and provide advanced configuration options.
- **Multi-AP setups:** For seamless roaming, consider pairing a MikroTik router with Ubiquiti APs.
- **Hands-on learning:** MikroTik is excellent for exploring routing, VLANs, firewalling, and Layer 3 networking.
