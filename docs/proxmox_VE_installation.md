
# Proxmox VE Installation Guide

## Overview

This guide walks through:

- Installing Proxmox VE on a workstation
- Performing the initial setup
- Accessing the Proxmox web interface

---

# 1. Install Proxmox VE on the machine

## Download Proxmox VE ISO

Download the latest Proxmox VE ISO from the official website:

https://www.proxmox.com/en/downloads

---

## Create a Bootable USB

Use one of the following tools to create a bootable USB drive:

- Rufus
- Balena Etcher
- Ventoy

I used Balena Etcher.

Recommended USB size: **8 GB or larger**

---

## Boot from USB

1. Insert the bootable USB into the machine
2. Power on the system
3. Press `F12` repeatedly during startup
4. Select the USB device from the boot menu

---

## Start the Proxmox Installer

1. Select **Install Proxmox VE**
2. Accept the license agreement
3. Continue through the installer

---

## Select Installation Disk

On the machine, the installation target is typically:

- NVMe SSD
- SATA SSD (if installed)

Choose the correct disk carefully, as it will be formatted during installation.

---

## Set Root Password and Email

Configure:

- **Root Password**
  - Used for SSH and web login
- **Email Address**
  - Used for system notifications and alerts

Example:

```text
Username: root
````

---

## Configure Networking

Recommended configuration:

* Static IP address
* Internet connectivity enabled

Example network configuration:

```text
IP Address: 192.168.1.50
Subnet: /24
Gateway: 192.168.1.1
DNS: 1.1.1.1
```

Using a static IP prevents the Proxmox management interface from changing addresses after reboot.

---

# 2. First Boot & Accessing Proxmox

After installation completes, the system will reboot into the Proxmox console.

You will see a black terminal login screen.

---

## Login to the Console

Use the credentials configured during installation:

```text
Username: root
Password: <your-password>
```

---

## Verify Network Interfaces

Run the following command:

```bash
ip a
```

This displays:

* Network interfaces
* Assigned IP addresses
* Bridge interfaces (typically `vmbr0`)

Locate the IP address assigned to the Proxmox host.

---

## Access the Proxmox Web UI

From another device on the same network, open a browser and navigate to:

```text
https://<your-proxmox-ip>:8006
```

Example:

```text
https://192.168.1.50:8006
```

You may receive a browser security warning due to the default self-signed certificate.

Proceed to the site and login using:

```text
Username: root
Realm: Linux PAM authentication
Password: <your-password>
```

---

# Expected Result

You should now have:

* A working Proxmox VE installation
* Web-based management access
* Network connectivity configured
* A ready platform for VMs, LXC containers, and services like Immich


