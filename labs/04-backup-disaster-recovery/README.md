# Project 04 – Backup & Disaster Recovery

## Overview

In this project, I built an automated Linux backup and recovery solution using Red Hat Enterprise Linux 10.2.

The purpose was to learn how Linux administrators protect application data, automate routine backups, troubleshoot failures and restore files after accidental deletion.

I also wanted to gain practical experience with Bash scripting, rsync, NFS, systemd and SELinux.

## Lab Environment

| Server | Role | IP address |
|---|---|---|
| rhel-app01 | Application server | 192.168.100.10 |
| rhel-storage01 | Backup storage server | 192.168.100.20 |

Both servers run RHEL 10.2 in Oracle VirtualBox.

**Backup source:** `/appdata/backup-demo/`

**Backup destination:** `/mnt/nfs/shared/backups/project04/`

The backup destination is mounted from `rhel-storage01` using NFSv4.2.

## 1. Creating the Backup Environment

Created a dedicated application directory on the LVM filesystem:

```bash
sudo mkdir -p /appdata/backup-demo
sudo chown linuxadmin:linuxadmin /appdata/backup-demo
```

Created simulated application data:

```bash
echo "APP_ENV=production" > /appdata/backup-demo/app.conf
echo "Customer database backup - version 1" > /appdata/backup-demo/database.txt
```

Prepared the backup destination on the storage server:

```bash
mkdir -p /srv/nfs/shared/backups/project04
```

## 2. Performing a Manual Backup

Used `rsync` to transfer the application data to the NFS storage server:

```bash
rsync -av /appdata/backup-demo/ /mnt/nfs/shared/backups/project04/
```

Verified that both files were available and readable on `rhel-storage01`.

## 3. Testing Disaster Recovery

Simulated accidental deletion of an application configuration file:

```bash
rm -i /appdata/backup-demo/app.conf
```

Restored the deleted file from the NFS backup:

```bash
rsync -av /mnt/nfs/shared/backups/project04/app.conf /appdata/backup-demo/
```

Verified the restored contents:

```text
APP_ENV=production
```

**Result:** Successfully recovered the deleted configuration file without affecting the remaining application data.

## 4. Automating Backups Using Bash

Created a Bash script at:

`/home/linuxadmin/bin/backup-project04.sh`

```bash
#!/bin/bash
set -euo pipefail

SOURCE="/appdata/backup-demo/"
DEST="/mnt/nfs/shared/backups/project04/"

echo "Starting backup..."

# Trigger the NFS automount
ls "$DEST" > /dev/null

# Verify the backup destination is mounted over NFS
if ! findmnt -rn -T "$DEST" -t nfs4 > /dev/null; then
    echo "ERROR: NFS backup destination is not mounted!"
    exit 1
fi

rsync -av "$SOURCE" "$DEST"

echo "Backup completed successfully!"
```

The script checks that the backup destination is an NFS filesystem before executing rsync.

This helps prevent accidentally writing backup files to a local directory if the network filesystem is not mounted.

## 5. Troubleshooting SELinux

One of the most valuable parts of this project was troubleshooting a failed systemd backup service.

The Bash script worked when executed manually but failed when started as a system-wide systemd service.

The service journal showed:

```text
rsync: change_dir "/appdata/backup-demo" failed: Permission denied (13)
rsync: cannot stat destination: Permission denied (13)
```

Investigated the problem using:

```bash
journalctl -u backup-project04.service
ausearch -m AVC,USER_AVC -ts recent
ls -Zd /appdata /appdata/backup-demo
```

Discovered that the application data had an incorrect SELinux label:

`unlabeled_t`

Configured persistent SELinux labels using `semanage fcontext` and applied them using `restorecon`:

```bash
sudo semanage fcontext -a -t rsync_data_t '/appdata'
sudo semanage fcontext -a -t rsync_data_t '/appdata/backup-demo(/.*)?'
sudo restorecon -v /appdata
sudo restorecon -Rv /appdata/backup-demo
```

Enabled the `rsync_client` SELinux boolean:

```bash
sudo setsebool -P rsync_client on
```

This resolved the observed source-directory denial, but the system-wide service still encountered restrictions accessing the home directory and NFS destination.

I investigated an alternative approach using a systemd user service. This worked without disabling SELinux.

**Lesson learned:** Linux filesystem permissions and SELinux are separate security mechanisms. Troubleshooting requires examining both.

## 6. Configuring a systemd User Service

Created:

`~/.config/systemd/user/backup-project04.service`

```ini
[Unit]
Description=Project 04 - Automated NFS Backup

[Service]
Type=oneshot
ExecStart=/usr/bin/bash /home/linuxadmin/bin/backup-project04.sh
```

Successfully tested the service through the systemd user manager.

The service completed without errors.

## 7. Scheduling Automatic Backups

Created:

`~/.config/systemd/user/backup-project04.timer`

```ini
[Unit]
Description=Daily Backup - Project 04

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
Unit=backup-project04.service

[Install]
WantedBy=timers.target
```

Configured the timer to run daily at 02:00.

Enabled user lingering:

```bash
sudo loginctl enable-linger linuxadmin
```

Verified the configuration:

```text
Linger=yes
Timer enabled: enabled
Timer status: active
```

The timer is scheduled, but an actual unattended scheduled execution still needs to be verified.

## 8. Verification Results

| Test | Result |
|---|---|
| Manual rsync backup | Passed |
| Files readable from storage server | Passed |
| Simulated file deletion | Passed |
| File restoration | Passed |
| Restored file contents | Verified |
| Bash backup script | Passed |
| SELinux troubleshooting | Completed |
| systemd user service | Passed |
| Timer syntax validation | Passed |
| Timer enabled and active | Verified |
| User lingering | Enabled |
| Unattended scheduled execution | Not yet tested |

## Limitations and Future Improvements

This is a learning environment, not a complete production disaster recovery solution.

The current rsync script maintains a single synchronized copy of the data. It does not preserve previous versions when existing files are overwritten.

Both virtual machines are hosted on the same physical computer, so a failure of that computer could affect both the original data and its backup.

Future improvements include:

- Versioned backups and retention policies
- Checksum-based backup verification
- Failure notifications and monitoring
- Automated restore testing
- Backups stored on independent physical or off-site storage

## Skills Demonstrated

Linux administration, Bash scripting, rsync, NFS, LVM, XFS, filesystem permissions, SELinux troubleshooting, journalctl, systemd services, systemd timers and file recovery.

## Why I Built This Project

I'm building this lab to prepare for a career in Linux system administration.

I want to develop practical skills by configuring systems myself, understanding how they work, troubleshooting problems and documenting what I've learned.

This project helped me understand an important principle:

**A backup is not fully validated until you have successfully restored data from it.**

The goal of this repository is to demonstrate my technical development, problem-solving approach and commitment to learning Linux infrastructure administration.
