# Project 02 – NFS Network Storage

## Overview

Configured a two-server NFSv4.2 environment using Red Hat Enterprise Linux 10.2 and Oracle VirtualBox.

The objective was to provide centralized network storage to an application server, implement controlled access, and configure persistent mounting using systemd.

## Lab Architecture

| Server | Role | Private IP |
|---|---|---|
| rhel-storage01 | NFS server | 192.168.100.20 |
| rhel-app01 | NFS client | 192.168.100.10 |

Both virtual machines use a dedicated VirtualBox Internal Network named `lab-nfs`, with a separate NAT adapter for internet access.

**Storage path:** `/srv/nfs/shared`

**Client mount point:** `/mnt/nfs/shared`

**Protocol:** NFSv4.2 over TCP port 2049

## Implementation

### 1. Network configuration

- Configured static IPv4 addresses using NetworkManager (`nmcli`).
- Used a private interface (`enp0s8`) on each VM.
- Verified bidirectional connectivity using `ping`.

### 2. NFS server

- Installed `nfs-utils`.
- Enabled and started `nfs-server.service`.
- Created the shared directory `/srv/nfs/shared`.
- Configured `/etc/exports` to authorize only the application server.

Export configuration:

`/srv/nfs/shared 192.168.100.10(rw,sync,root_squash,no_subtree_check)`

Activated the export using `exportfs -rav` and verified it using `exportfs -v`.

### 3. Firewall configuration

Created a dedicated firewalld zone named `nfs-lab` and assigned it to the private interface `enp0s8`.

Restricted NFS access to TCP port 2049 from `192.168.100.10/32`.

The NAT interface remained in the default `public` zone.

### 4. NFS client

- Installed `nfs-utils` on `rhel-app01`.
- Created `/mnt/nfs/shared`.
- Mounted the remote export using NFSv4.
- Verified the mount using `findmnt`.

### 5. File permissions

Initially, a normal user could not write to the shared directory because it belonged to root.

Verified UID and GID mappings on both servers:

- `linuxadmin`: UID 1000, GID 1000
- `linuxstorage`: UID 1000, GID 1000

Changed directory ownership to `linuxstorage:linuxstorage` and successfully verified remote write access.

Kept `root_squash` enabled to prevent remote root from automatically receiving root privileges on the export.

### 6. Persistent NFS mounting

Configured `/etc/fstab` on the client:

`192.168.100.20:/srv/nfs/shared /mnt/nfs/shared nfs4 rw,_netdev,nofail,x-systemd.automount 0 0`

Used systemd automount to establish the NFS connection on demand.

Validated the configuration using `findmnt --verify --verbose`.

## Verification Results

| Test | Result |
|---|---|
| Bidirectional ICMP connectivity | Passed |
| NFS export validation | Passed |
| NFSv4.2 mount | Passed |
| Server-to-client file reading | Passed |
| Client-to-server file creation | Passed |
| UID/GID-based permissions | Passed |
| Systemd automount | Passed |
| NFS access after client reboot | Passed |

## Troubleshooting

**Package installation failure:** `nfs-utils` was initially unavailable on the application server because the RHEL BaseOS and AppStream repositories were disabled. Enabled both repositories through Subscription Manager.

**Permission denied:** The client initially lacked write access because the export directory was owned by root. Resolved by configuring directory ownership for UID 1000.

## Security Notes

This lab uses NFS `sec=sys`, which relies on client-supplied UNIX user identities. It is appropriate for demonstrating functionality in a controlled, isolated lab, but does not provide strong client authentication or encryption.

A production deployment would require additional security planning, potentially including Kerberos authentication and stronger network controls.

## Skills Demonstrated

Linux system administration, NetworkManager, NFSv4, TCP/IP networking, firewalld, Linux permissions, UID/GID mapping, systemd automount, `/etc/fstab`, troubleshooting, and validation after reboot.
