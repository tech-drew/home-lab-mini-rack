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
