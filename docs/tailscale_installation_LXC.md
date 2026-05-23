# Setting up Tailscale inside the Immich LXC

This (installing tailscale on proxmox and LXC) is done so that Immich server will behave as a seperate device rather than pass-through via Proxmox.

## Verify existing Tailscale status

Enter the LXC:

```bash
pct enter 100
````

Run:

```bash
tailscale status
tailscale ip
```

This verifies whether Tailscale is already installed and connected.

---

# Install Tailscale inside the LXC

Official references:

https://tailscale.com/docs/install/linux
https://tailscale.com/kb/1130/lxc

Run inside the LXC:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

This installs:

* `tailscale`
* `tailscaled`

---

# Bring the node online

Run:

```bash
tailscale up
```

This begins the authentication process.

---

# Troubleshooting tailscaled startup issues

If you encounter issues related to `tailscaled`, run:

```bash
systemctl start tailscaled
```

Then retry:

```bash
tailscale up
```

---

# Required LXC configuration for Tailscale

For Tailscale networking support, ensure these entries exist in:

```text
/etc/pve/lxc/100.conf
```

Open the config on the Proxmox host:

```bash
nano /etc/pve/lxc/100.conf
```

Ensure the following lines exist:

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.auto: proc:rw sys:rw
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

These enable:

* TUN device access
* Networking support
* Proper Tailscale operation inside the LXC

---

# Convert the LXC to privileged mode

The container was ultimately converted to a:

```text
Privileged LXC
```

This resolved:

* `tailscaled` startup crashes
* TUN permission issues
* systemd-related networking problems

---

# Restart the LXC

Run on the Proxmox host:

```bash
pct stop 100
pct start 100
```

---

# Authenticate the LXC with Tailscale

Run inside the LXC:

```bash
tailscale up
```

Tailscale will generate a login URL similar to:

```text
https://login.tailscale.com/a/70abcedfghijk
```

Open the URL in a browser and authenticate the node into your Tailnet.

---

# Verify the node in the Tailscale dashboard

After authentication:

* Open the Tailscale admin dashboard
* Verify the Immich LXC appears as an active node

You can now retrieve:

* Tailnet hostname
* Assigned Tailscale IP
* MagicDNS hostname

---

# Access Immich through Tailscale

Immich default port:

```text
2283
```

Example access URLs:

```text
http://100.x.x.x:2283
```

Or using MagicDNS:

```text
http://immich-lxc.tailnet-name.ts.net:2283
```

When opened successfully, Immich will display the initial setup page.

---

# Complete Immich initial setup

Complete:

* Admin account creation
* Storage configuration
* User setup
* Mobile backup preferences

---

# Install the mobile apps

Install:

* Immich mobile app
* Tailscale mobile app

on your mobile phone.

---

# Authenticate Tailscale on the mobile

Log into the same Tailnet from the mobile device using the Tailscale app.

This enables secure private access to the Immich server without port forwarding.

---

# Connect the Immich mobile app

Inside the Immich mobile app, use:

```text
http://<tailscale-host-or-ip>:2283
```

Example:

```text
http://immich-lxc.tailnet-name.ts.net:2283
```

Authenticate using the Immich credentials created during setup.

---

# Final Result

At this stage, the setup provides:

* Private remote access via Tailscale
* No public port forwarding required
* Secure media synchronization
* Automatic mobile photo/video backup
* Full Immich media management functionality
