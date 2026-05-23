# Setting Up LXC Storage on Proxmox

## Install LXC

1. Open the Proxmox Web UI
2. Navigate to:

```text
Datacenter → <Your Node> → CT Templates
````

3. Download a Debian template

> Recommended (Debian better than Ubuntu in terms of setting up config)

```text
Debian 13 or the latest stable version
```

4. Create a new LXC container using the downloaded template

---

# Prepare dedicated storage for LXC

## Reformat the SSD

Reformat the SSD planned for LXC/container storage using:

```text
ext4
```

> Warning: This will erase all existing data on the drive.

---

## Create a mount point

Run the following command on the Proxmox host:

```bash
mkdir -p /mnt/ssd1
```

This creates the mount directory for the SSD.

---

## Find the disk ID

Instead of mounting using `/dev/sdX`, use the persistent disk ID. This prevents identifying the disk on each start due to the drive getting assigned with different label.

Run:

```bash
ls -la /dev/disk/by-id
```

Locate the correct SSD identifier.

Example:

```text
nvme-MSI_M470_PRO_1TB_511251002084004416-part1
```

---

## Manually mount the SSD

Example command:

```bash
mount /dev/disk/by-id/nvme-MSI_M470_PRO_1TB_511251002084004416-part1 /mnt/ssd1
```

This mounts the SSD temporarily.

---

# Configure persistent mount using fstab

## Open the fstab file

Run:

```bash
nano /etc/fstab
```

> Execute this command on the Proxmox host node.

---

## Add the mount entry

Add the following line to the end of the file:

```fstab
/dev/disk/by-id/nvme-MSI_M470_PRO_1TB_511251002084004416-part1  /mnt/ssd1  ext4  defaults,noatime,discard  0  2
```
> Remember to change the disk ID found in previous step.

### Option explanation

| Option   | Description              |
| -------- | ------------------------ |
| defaults | Standard mount options   |
| noatime  | Reduces disk writes      |
| discard  | Enables SSD TRIM support |

---

## Validate the fstab configuration

Run:

```bash
mount -a
```

This will:

* Mount all configured filesystems
* Show errors immediately if configuration is invalid

---

## Confirm the SSD is mounted

Run:

```bash
df -h | grep /mnt/ssd1
```

Expected output should display:

* Mounted filesystem
* Storage capacity
* Usage statistics
* Mount location

---

## View all mounted devices

Run:

```bash
mount
```

This lists:

* All mounted drives
* Filesystem types
* Mount options

---

# Reboot the Proxmox host

Recommended after storage configuration:

```bash
Click on the Reboot button on the proxmox node.
```

After reboot, verify the SSD mounts automatically.

---

# Optional — Install duf for better disk visibility

`duf` provides a cleaner and easier-to-read storage overview.

## Install duf

Run:

```bash
apt update && apt install duf -y
```

> Execute this command on the Proxmox host node.

---

## Run duf

```bash
duf
```

This displays:

* Mounted devices
* Filesystem types
* Used/free space
* Mount points
* Usage percentages
