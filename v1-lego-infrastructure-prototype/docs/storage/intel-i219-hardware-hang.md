# Intel-I219-LM-Hardware-Hang

In Linux and FreeBSD environments (which power Proxmox Backup Server and TrueNAS), the driver for this card (e1000e) suffers from a severe flaw known as a "Detected Hardware Unit Hang". The HP Elitedesk G4 SFF's used in this environment suffer from this issue. The information below describes the error in more detail and how to fix it.

## The Fix
The hardware lockup is resolved by **disabling TCP Segmentation Offload (TSO) and Generic Segmentation Offload (GSO)**. This forces the system CPU to process packet chunking instead of offloading it to the unstable hardware ring buffer on the physical Intel network interface card (NIC). This fix only needs to be applied to the primary physical interface (`eth0`/`eno1`); virtual VLANs or bridge interfaces inherit the protection automatically.

## Why This is Happening

When the server undergoes heavy network load—such as running massive backups to PBS or moving files over TrueNAS—the NIC's internal transmit descriptor ring gets stuck. The use of Jumbo Frames and VLANs heavily exacerbates this bug, as it stresses the hardware offloading features.
Because it is a driver-level/hardware-level lockup rather than a physical link disconnection:
• The physical switch port and NIC LEDs will continue to show activity and link status.
• The operating system thinks the interface is still UP, but it will stop routing any actual data packets, rendering it completely unpingable and unreachable.

## How to Confirm It

If you connect a monitor directly to the EliteDesk boxes or view the local shell when they drop off, running the following command will show rows of errors like e1000e: Detected Hardware Unit Hang and Reset adapter unexpectedly:

```bash
dmesg | grep -i "e1000e\|hang"
```

## Proxmox Backup Server (PBS) Fix

Permanent fix: Edit your network configuration file at `/etc/network/interfaces` and add a post-up hook under your primary network interface:

```text
iface eno1 inet manual
      post-up /sbin/ethtool -K eno1 tso off gso off
```

Below is an example of the working config in use on a Proxmox Backup Server.

```text
auto lo
iface lo inet loopback

# Physical NIC
auto eth0
iface eth0 inet manual
    mtu 9000
    post-up sleep 2 && /sbin/ethtool --set-eee eth0 eee off
    post-up sleep 2 && /sbin/ethtool -s eth0 wol d
    post-up sleep 2 && /sbin/ethtool -K eth0 tso off gso off
    # This is the single physical NIC on the node. MTU 9000 enables jumbo frames.
    # The set-eee eth0 off turns off low power mode. If the setting is enabled your nic may turn off and you can lose device connectivity.
    # The sleep 2 avoids a race condition upon proxmox reboot
    # The tso off gso off disables segmentation offloading to prevent the e1000e "Hardware Unit Hang" bug.
    # All virtual bridges and VLAN interfaces will use this NIC as the base.

# Virtual bridge connecting VMs, VLANs, and storage
auto vmbr0
iface vmbr0 inet manual
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
    mtu 9000
    # This bridge is VLAN-aware, allowing multiple VLANs over the same physical NIC.
    # All VM and storage traffic passes through this bridge.

# Backup interface (VLAN 50)
auto vmbr0.50
iface vmbr0.50 inet static
    address 10.100.50.16/24
    mtu 9000
    # Dedicated interface for backups
    # Keep backup traffic separate from management to improve performance and security.

# Management interface (VLAN 99)
auto vmbr0.99
iface vmbr0.99 inet static
    address 10.100.99.16/24
    gateway 10.100.99.1
    mtu 1500
    # This interface is used for Proxmox management, web GUI, SSH, and cluster communication.
    # The web GUI and other admin services should only be accessible on this VLAN.

source /etc/network/interfaces.d/*
```

## TrueNAS (SCALE or CORE) Fix

Because configuration files wipe on software updates in TrueNAS, the parameters must be set via a persistent **Post-Init Script** in the Web GUI.

### Step 1: Open the Init/Shutdown Scripts Menu

1. On the left-hand navigation pane of your TrueNAS dashboard, click on **System** (at the bottom of the list).
2. In the expanded submenu, select **Advanced**.
3. Scroll down until you locate the **Init/Shutdown Scripts** card, then click the **Add** button.

### Step 2: Configure the Script

Fill out the configuration panel with the following parameters exactly:
• **Description:** Disable eno1 TSO/GSO network hang fix
• **Type:** Select **Command** from the dropdown menu.
• **Command:** Enter the following line exactly to target your physical card (`eno1`):
```bash
/usr/sbin/ethtool -K eno1 tso off gso off
```
• **When:** Select **Post Init**.
• **Enabled:** Check the box.

### Step 3: Immediate Live Application (Optional)
To avoid a system reboot, you can apply the command instantly via the TrueNAS Shell or SSH:
```bash
ethtool -K eno1 tso off gso off
```
