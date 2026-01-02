# Basic SSH Setup Guide for Homelab Nodes

This guide covers setting up SSH on Kubuntu minimal installs in your homelab, allowing remote access for administration and study purposes.

---

## 1. Update Package Lists
Always start by updating the package lists to ensure you have the latest versions:

```bash
sudo apt update
````

---

## 2. Install OpenSSH Server

Install the OpenSSH server package to allow incoming SSH connections:

```bash
sudo apt install openssh-server
```

---

## 3. Enable and Start SSH Service

Enable SSH to start automatically on boot and start the service immediately:

```bash
sudo systemctl enable --now ssh
```

You can check the status of the SSH service with:

```bash
systemctl status ssh
```

---

## 4. Configure Firewall (Optional but Recommended)

If you have `ufw` enabled, allow SSH connections through the firewall:

```bash
sudo ufw allow ssh
sudo ufw enable     # if not already enabled
sudo ufw status
```

---

## 5. Verify SSH Access

From another machine, you can test the connection using:

```bash
ssh username@node_ip_address
```

Replace `username` with your user account and `node_ip_address` with the IP of your Kubuntu node.

---

## Notes

* The SSH **client** is installed by default, so you can always connect *out* of the node.
* The SSH **server** must be installed and running for remote access.
* This setup is sufficient for lightweight homelab nodes running minimal Kubuntu installs.

```

```

# Configuring Hostname and SSH Access by Name

This guide explains how to configure the hostname for a Kubuntu homelab node and how to SSH into it using that hostname.

---

## 1. Set or Change the Hostname

On Kubuntu 24.04 (and most modern Ubuntu variants), use `hostnamectl` to set the hostname:

```bash
sudo hostnamectl set-hostname new-hostname
````

* Replace `new-hostname` with your desired hostname (e.g., `kube-node-01`).
* This updates the **static hostname** and persists across reboots.

Verify the hostname with:

```bash
hostnamectl status
```

or simply:

```bash
hostname
```

---

## 2. Update `/etc/hosts` (Optional but Recommended)

Updating `/etc/hosts` ensures local services can resolve the hostname even without DNS:

```bash
sudo nano /etc/hosts
```

Add a line like:

```
127.0.1.1   new-hostname
```

Save and exit the file.

---

## 3. SSH Using the Hostname

Whether you can SSH using the hostname depends on name resolution:

* **With DNS**: If your network provides DNS (e.g., via your router) and registers hostnames, you can SSH directly:

```bash
ssh username@new-hostname
```

* **Without DNS**: You can either:

  1. SSH via the node’s IP address:

```bash
ssh username@192.168.1.50
```

2. Or add a local `/etc/hosts` entry on the client machine mapping hostname → IP:

```
192.168.1.50   new-hostname
```

Then SSH works using the hostname:

```bash
ssh username@new-hostname
```

---

## Notes

* Setting the hostname helps identify nodes in a homelab cluster.
* In networks without DNS, maintaining a `/etc/hosts` mapping for all nodes is recommended.
* For homelab setups with multiple nodes, SSHing by hostname improves readability over using IP addresses.

```
```
