# Maintenance

Regular maintenance helps keep the Proxmox host, LXC container, Docker services, Immich, and Tailscale up to date.

> **Important:** Always note which environment a command should be executed in. 
>                Commands for the Proxmox host should not be run inside the Immich LXC unless explicitly stated.

---

## 1. Update Proxmox Host

### Using the Proxmox Web UI

1. Open the Proxmox Web UI.
2. Select the Proxmox node.
3. Go to **Updates**.
4. Click **Refresh**.
5. Review the available updates.
6. Click **Update**.

### Using the Terminal

Run on the **Proxmox host**:

```bash
apt update
apt full-upgrade
````

Review the proposed changes before confirming the upgrade.

---

# 2. Update the Immich LXC

The LXC operating system should also be kept up to date.

Entering the immich lxc:

a)Enter the LXC from the Proxmox host:

```bash
pct enter <containerID>
```
b)Navigate to the console inside the immich LXC, under proxmox node in the web UI.

Then run inside the **Immich LXC**:

```bash
apt update
apt full-upgrade
```

If the kernel or other major system components are updated, reboot the LXC when appropriate.

From proxmox shell:
```bash
pct stop <containerID>
pct start <containerID>
```

---

# 3. Update Immich

Immich is deployed using Docker Compose.

Enter the Immich LXC using either of the options mentioned before.

Go to the Immich installation directory:

```bash
cd /opt/immich
```

Stop the current Immich containers:

```bash
docker compose down
```

Download the latest Immich container images:

```bash
docker compose pull
```

Start Immich again:

```bash
docker compose up -d
```

---

## Verify Immich Status

Run inside the **Immich LXC**:

```bash
docker compose ps
```

Check that the Immich services are running.

---

# 4. Clean Up Old Docker Images

After successfully updating Immich, unused Docker images can be removed.

Run inside the **Immich LXC**:

```bash
docker image prune -f
```

This removes unused Docker images that are no longer referenced by containers.

> Run this only after confirming that the updated Immich stack is working correctly.

---

# 5. Update Tailscale

Tailscale may be installed on both:

* Proxmox host
* Immich LXC

Update it on both systems.

## Proxmox Host

Run on the **Proxmox host**:

```bash
apt update
apt install tailscale
```

---

## Immich LXC

Enter the LXC.

Then run inside the **Immich LXC**:

```bash
apt update
apt install tailscale
```

Verify the Tailscale connection:

```bash
tailscale status
```

And check the assigned Tailscale IP:

```bash
tailscale ip
```

---

# 6. Reboot After Maintenance

A reboot can be useful after major system, Docker, networking, or Tailscale updates.

## Reboot Proxmox Host

(Optional) Run on the **Proxmox host**:

```bash
pct stop <containerID>
```
Press the **reboot** button on the web UI and wait for Proxmox to become available again.

---

## Verify the Immich LXC

From the Proxmox host:

(Optional) If the container was stopped, then:

```bash
pct start <containerID>
```
Then execute:
```bash
pct status <containerID>
```
Expected:

```text
status: running
```

If necessary, enter the LXC and verify Immich:

```bash
cd /opt/immich
docker compose ps
```

---

# 7. Enable Mobile Backup Over Cellular

If mobile photo/video backups should also work when the phone is not connected to a Wi-Fi:

Open the **Immich mobile app**.

Navigate to:

```text
Settings → Backup → Videos/Photos backup over cellular
```

Enable the required cellular backup options.

> Cellular backup can use significant mobile data, particularly for video libraries.

---

# 8. Troubleshooting LXC / Docker Issues

If an update causes problems or the LXC behaves unexpectedly, first check the systemd state.

Enter the LXC:

```bash
pct enter <containerID>
```

Run:

```bash
systemctl status
```

Look for an overall state such as:

```text
State: degraded
```

---

## Find Failed Services

If the system is degraded, run:

```bash
systemctl --failed
```

This lists services that failed to start.

Example:

```text
UNIT                 LOAD   ACTIVE SUB    DESCRIPTION
some-service.service loaded failed failed Some Service
```

---

## Investigate the Failed Service

Use the service name reported by `systemctl --failed`.

For example:

```bash
systemctl status <service-name>
```

Check the output for:

* Configuration errors
* Permission problems
* Missing files
* Dependency failures
* Networking issues
* Failed mounts

Make the necessary correction based on the reported error.

---

# 9. Post-Maintenance Checklist

After completing maintenance, verify the following:

* [ ] Proxmox host is updated
* [ ] Immich LXC is updated
* [ ] Immich Docker images are updated
* [ ] Immich containers are running
* [ ] Immich web interface is accessible
* [ ] Immich storage is mounted at `/mnt/immich`
* [ ] Tailscale is connected on the required systems
* [ ] Mobile Immich app can connect
* [ ] Photo/video backup is working
* [ ] No unexpected failed systemd services

---

# Recommended Maintenance Order

For a normal maintenance cycle, use this order:

```text
1. Proxmox host
       ↓
2. Immich LXC
       ↓
3. Immich Docker containers
       ↓
4. Docker image cleanup
       ↓
5. Tailscale
       ↓
6. Reboot
       ↓
7. Verify services
       ↓
8. Verify Immich mobile backup
```

This provides a simple repeatable maintenance procedure for the Proxmox + Immich server.

```
```
