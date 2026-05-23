# Setting up Immich on the Proxmox LXC

Refer official Immich website as well:

https://immich.app/

---

# Create Immich media directory on Proxmox host

Run the following command on the **Proxmox host**:

```bash
mkdir -p /mnt/ssd1/immich
````

This directory will store all uploaded Immich media.

---

# Bind-mount the media directory into the LXC

Open the LXC configuration file:

```bash
nano /etc/pve/lxc/100.conf
```

> Execute this command on the Proxmox host node.

---

## Add the mount point

Add the following line:

```ini
mp0: /mnt/ssd1/immich,mp=/mnt/immich
```

### What this means?

| Host Path          | LXC Path                        |
| ------------------ | ------------------------------- |
| `/mnt/ssd1/immich` | Physical SSD storage on Proxmox |
| `/mnt/immich`      | Path visible inside the LXC     |

Docker and Immich inside the container will use `/mnt/immich`.

---

# Restart the LXC

Run the following commands on the Proxmox host:

```bash
pct stop 100
pct start 100
```

---

# Verify the mount inside the LXC

Enter the LXC:

```bash
pct enter 100
```

Then run:

```bash
ls /mnt/immich
```

If successful, the directory should exist.

---

# Troubleshooting missing mounts

If you see:

```text
ls: cannot access '/mnt/immich': No such file or directory
```

Check whether the container is unprivileged:

```bash
grep unprivileged /etc/pve/lxc/100.conf
```

> Execute this command on the Proxmox host node.

---

# Configure UID/GID mapping (if needed)

Check ownership of the Immich directory:

```bash
stat -c "%u %g" /mnt/ssd1/immich
```

Typical output:

```text
0 0
```

---

## Update the LXC configuration

Open the container config again:

```bash
nano /etc/pve/lxc/100.conf
```

Add:

```ini
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 65536
```

Save and exit.

---

# Restart the LXC again

Run:

```bash
pct stop 100
pct start 100
```

---

# Verify the mount from inside the LXC

Enter the container:

```bash
pct enter 100
```

Check available mounts:

```bash
ls /mnt
```

Expected output:

```text
immich
```

---

## Verify active mount

Run:

```bash
mount | grep immich
```

Expected result:

* `/mnt/immich` listed
* Filesystem type shown (`ext4`)

If unsuccessful, you may see:

```text
findmnt: /mnt/immich: not a mountpoint
```

---

# Verify the mount from the Proxmox host

Exit the LXC:

```bash
exit
```

Then run on the Proxmox host:

```bash
ls -ld /mnt/ssd1/immich
```

This confirms the folder exists correctly on the SSD.

---

# Restart the LXC before Immich installation

Run:

```bash
pct stop 100
pct start 100
```

---

# Prepare Immich installation directory

Enter the LXC:

```bash
pct enter 100
```

Create the installation directory:

```bash
mkdir -p /opt/immich
```

Move into it:

```bash
cd /opt/immich
```

---

# Download Immich docker compose file

Run inside the LXC:

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```

---

# Download the environment File

Run:

```bash
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

---

# Configure Immich environment variables

Open the environment file:

```bash
nano /opt/immich/.env
```

Update the following:

## Upload Location

Change:

```env
UPLOAD_LOCATION=./library
```

To:

```env
UPLOAD_LOCATION=/mnt/immich
```

---

## Timezone

Uncomment and update:

```env
TZ=<local-timeZone>
```

---

## Database password

Replace the default database password with a stronger **alphanumeric** password.

Example:

```env
DB_PASSWORD=StrongPassword123
```

---

# Start Immich using docker compose

Run inside the LXC:

```bash
docker compose up -d
```

Wait for all containers to initialize.

---

# Access the Immich web UI

Retrieve the LXC IP address from:

```text
Proxmox Web UI → LXC Container → Summary
```

Immich default port:

```text
2283
```

Example:

```text
http://192.168.1.1:2283
```

If the page loads successfully, the deployment is working.

---

# Verify docker containers

Run:

```bash
docker compose ps
```

This lists:

* Running containers
* Container status
* Port mappings

---

# Check container logs

To inspect logs:

```bash
docker logs <resource_name> --tail=50
```

Example:

```bash
docker logs immich_server --tail=50
```

---

# Troubleshooting permission issues

Sometimes Immich cannot write to the upload directory because of UID/GID mapping.

Typical symptoms:

* Upload failures
* Permission denied errors
* Missing generated thumbnails

---

## Verify mount availability

Run inside the LXC:

```bash
ls -l /mnt
```

Expected result:

```text
immich
```

---

# Fix folder permissions

Depending on the UID/GID mapping setup, permissions may need adjustment on the Proxmox host.

Example fix:

```bash
chown -R 101000:101000 /mnt/ssd1/immich
```

> Execute this command on the Proxmox host node.

This grants the unprivileged container permission to write to the mounted storage.

---

# Restart Immich containers

Run inside the LXC:

```bash
docker compose down
docker compose up -d
```

Verify status:

```bash
docker compose ps
```

---

# Final verification

Once all services are healthy, Immich should be accessible from:

```text
http://<lxc-ip>:2283
```

Example:

```text
http://192.168.1.1:2283
```

You can now **over LAN**:

* Upload photos and videos
* Enable mobile backups
* Configure users and albums
* Use AI-powered search and facial recognition
