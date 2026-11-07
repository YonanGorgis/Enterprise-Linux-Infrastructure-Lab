# RHEL 10.2 — LVM Storage Configuration & Online XFS Expansion

**Part of:** Enterprise Linux Infrastructure Lab  
**Environment:** Red Hat Enterprise Linux 10.2 (Minimal Install), Oracle VirtualBox  
**Focus:** Linux storage administration, logical volume management, filesystems, persistent mounts, verification

## Overview

This hands-on lab demonstrates how to provision a new virtual disk on a RHEL server, configure it with Logical Volume Manager (LVM), create an XFS filesystem, mount it persistently, and extend the volume and filesystem online while preserving existing test data.

**Result:** A new `/appdata` filesystem, expanded from **5 GiB** to **8 GiB**, mounted from `/dev/vg_data/lv_appdata`. The test file remained readable after the expansion. The mount was also confirmed after reboot.

## Environment and topology

- **Host:** Windows 11, 16 GB RAM
- **Hypervisor:** Oracle VirtualBox
- **Virtual machine:** `rhel-app01`
- **Guest OS:** RHEL 10.2, Minimal Install
- **VM resources:** 2 vCPU, 4 GB assigned RAM
- **System disk:** `/dev/sda` (50 GB), containing the operating system
- **Additional lab disk:** `/dev/sdb` (10 GB), dedicated to this LVM exercise
- **Management:** SSH; remote access to the lab was additionally configured through Tailscale

```text
VirtualBox VM: rhel-app01
├── /dev/sda   50G  (existing RHEL system disk — unchanged)
│   ├── /dev/sda1
│   ├── /dev/sda2             XFS /boot
│   └── /dev/sda3             LVM PV, VG: rhel
│       ├── /dev/rhel/root    ~44.1G  XFS /
│       └── /dev/rhel/swap    ~3.9G   swap
└── /dev/sdb   10G  (new lab disk)
    └── LVM Physical Volume
        └── Volume Group: vg_data (~10G)
            └── Logical Volume: lv_appdata (5G → 8G)
                └── XFS filesystem mounted at /appdata
```

> **Scope / safety:** All destructive operations below were performed on the added, empty lab disk (`/dev/sdb`) or on its dedicated logical volume. **Do not run `pvcreate` or `mkfs.xfs` against an existing system or data disk.** Disk device names can differ between machines.

## Implementation

### 1. Identify the new disk

After attaching a second 10 GB VDI disk in VirtualBox:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

Verified that `/dev/sdb` was the new 10 GB device with no partitions, filesystem, or mount points.

### 2. Create the LVM physical volume and volume group

```bash
sudo pvcreate /dev/sdb
sudo vgcreate vg_data /dev/sdb
sudo pvs
sudo vgs
```

The new volume group `vg_data` initially had almost 10 GiB of free space.

### 3. Create a 5 GiB logical volume and XFS filesystem

```bash
sudo lvcreate -L 5G -n lv_appdata vg_data
sudo lvs
sudo mkfs.xfs /dev/vg_data/lv_appdata
```

### 4. Create the mount point and mount the filesystem

```bash
sudo mkdir -p /appdata
sudo mount /dev/vg_data/lv_appdata /appdata
findmnt /appdata
df -hT /appdata
```

### 5. Configure persistent mounting

Back up the configuration and retrieve the XFS UUID:

```bash
sudo cp -a /etc/fstab /etc/fstab.bak
sudo blkid /dev/vg_data/lv_appdata
```

Add **one line** to `/etc/fstab` using the UUID reported on this particular VM:

```fstab
UUID=<your-xfs-filesystem-uuid> /appdata xfs defaults 0 0
```

Verify the configuration and reload systemd:

```bash
sudo findmnt --verify --verbose
sudo systemctl daemon-reload
sudo mount -a
findmnt /appdata
```

The configuration check reported **0 parse errors, 0 errors, 1 warning**. The warning was about systemd using the previous version of the configuration; `systemctl daemon-reload` was then executed. After reboot, `findmnt /appdata` confirmed that the XFS volume was mounted automatically.

### 6. Write test data

```bash
echo "Enterprise Linux Infrastructure Lab - Storage Test" | sudo tee /appdata/storage-test.txt
cat /appdata/storage-test.txt
```

Verified output:

```text
Enterprise Linux Infrastructure Lab - Storage Test
```

### 7. Expand the logical volume online (5 GiB → 8 GiB)

Checked available VG space (approximately 5 GiB remained free), then expanded the existing logical volume:

```bash
sudo vgs vg_data
sudo lvextend -L 8G /dev/vg_data/lv_appdata
sudo lvs /dev/vg_data/lv_appdata
```

Next, expanded the **mounted XFS filesystem** to use the larger logical volume:

```bash
sudo xfs_growfs /appdata
```

The filesystem reported:

```text
data blocks changed from 1310720 to 2097152
```

### 8. Verify size and data integrity

```bash
df -hT /appdata
cat /appdata/storage-test.txt
sudo lvs -o lv_name,vg_name,lv_size,lv_path
```

Recorded verification:

```text
Filesystem                     Type  Size  Used Avail Use% Mounted on
/dev/mapper/vg_data-lv_appdata xfs   8.0G  189M  7.8G   3% /appdata
```

```text
Enterprise Linux Infrastructure Lab - Storage Test
```

```text
LV         VG      LSize   Path
root       rhel    <44.10g /dev/rhel/root
swap       rhel     <3.90g /dev/rhel/swap
lv_appdata vg_data   8.00g /dev/vg_data/lv_appdata
```

## Key takeaways

- **PV → VG → LV:** A physical volume contributes capacity to a volume group; logical volumes are allocated from that pool.
- **LVM vs. filesystem:** Enlarging the LV alone does not enlarge XFS. `xfs_growfs` is required to make the extra space usable.
- **Persistent mount:** An `/etc/fstab` entry based on the filesystem UUID allows it to mount at boot.
- **Verification matters:** `lsblk`, `pvs`, `vgs`, `lvs`, `findmnt`, `df`, and read-back checks confirm each stage.
- **Operational caution:** XFS supports online growth but not shrinking; LVM is not a substitute for RAID or backups.

## Next improvements

- Add sanitized terminal screenshots demonstrating the creation and successful expansion.
- Document a second storage scenario with permission-controlled access to `/appdata`.
- Continue the broader lab with SSH hardening, services, NFS, Bash automation, and controlled troubleshooting exercises.

## Notes

This is a self-directed **home lab**, not a production deployment or a claim of professional experience. The steps and validation outputs above were performed and recorded during the lab; where output is abbreviated, it is presented as an excerpt.
