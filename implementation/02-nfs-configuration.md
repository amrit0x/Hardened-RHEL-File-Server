# NFS Network File Sharing Configuration

## Objective

Configure Network File System (NFS) on RHEL to share the LVM-backed storage directory across the network.

The NFS server exports `/srv/nfs/shared`, allowing authorized clients to access the shared files through their local mount points.

## Architecture

```text
       NFS SERVER (RHEL)
       10.219.122.167
              |
              |
       /srv/nfs/shared
         (NFS Export)
              |
              |
        Network (NFSv4)
              |
              |
       NFS CLIENT (RHEL)
              |
              |
        /mnt/nfs/shared
        (Client Mount)
```

Changes made by an authorized client inside `/mnt/nfs/shared` are reflected in the server's shared directory. Files are accessed over the network rather than copied to the client.

## Technologies Used

- NFS (Network File System)
- nfs-utils
- firewalld
- AutoFS
- RHEL

## Implementation

### 1. Configure NFS Server

**Server IP:** `10.219.122.167`

#### Install NFS utilities

```bash
dnf install -y nfs-utils
```

#### Verify the storage mount

Confirmed that the LVM-backed filesystem is mounted at the intended export directory.

```bash
findmnt /srv/nfs/shared
df -hT /srv/nfs/shared
```

#### Create shared directory

```bash
mkdir -p /srv/nfs/shared
```

#### Configure NFS service

Enabled and started the NFS server:

```bash
systemctl enable --now nfs-server
```

#### Configure shared directory exports

Edited `/etc/exports`:

```bash
vim /etc/exports
```

Added the following export configuration:

```text
/srv/nfs/shared 10.219.122.0/24(rw,sync,root_squash,no_subtree_check)
```

**Export options:**

| Option             | Purpose                                               |
| ------------------ | ----------------------------------------------------- |
| `rw`               | Allows read and write access                          |
| `sync`             | Commits changes to stable storage before replying     |
| `root_squash`      | Maps remote root requests to an unprivileged identity |
| `no_subtree_check` | Disables subtree checking                             |

The export permits clients from the specified subnet. Appropriate Linux ownership and permissions are also required for write access.

#### Apply export configuration :&#x20;

```bash
exportfs -rav
```

#### Configure firewall :

Allowed NFS traffic through firewalld:

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
```

#### Verify NFS server :

```bash
exportfs -v

systemctl status nfs-server

firewall-cmd --list-services
```

### 2. Configure NFS Client

The client accesses the server's exported directory through a local mount point.

#### Install NFS utilities

```bash
dnf install -y nfs-utils
```

#### Create client mount point

```bash
mkdir -p /mnt/nfs/shared
```

#### Mount NFS share manually

```bash
mount -t nfs 10.219.122.167:/srv/nfs/shared /mnt/nfs/shared
```

#### Verify NFS mount:

```bash
df -hT /mnt/nfs/shared

mountpoint /mnt/nfs/shared

findmnt -t nfs,nfs4

ls -la /mnt/nfs/shared
```

#### Test file sharing

Created a test file from the client:

```bash
touch /mnt/nfs/shared/client-test.txt
```

Verified that the file was visible on the NFS server under:

```bash
ls -l /srv/nfs/shared/
```

This verifies that the client is accessing the server's shared storage, subject to the configured permissions.

#### Unmount NFS share

```bash
umount /mnt/nfs/shared
```

### 3. Configure Persistent NFS Mount Using /etc/fstab

Configured an alternative method for automatically mounting the NFS share during system startup.

Edited `/etc/fstab`:

```bash
vim /etc/fstab
```

Added:

```text
10.219.122.167:/srv/nfs/shared /mnt/nfs/shared nfs defaults,_netdev 0 0
```

**Verify configuration:**

```bash
mount -a

findmnt /mnt/nfs/shared

df -hT /mnt/nfs/shared
```

`_netdev` identifies the filesystem as network-dependent.

### 4. Configure AutoFS for On-Demand NFS Mounting

Configured AutoFS to mount the NFS share automatically when the client accesses the configured directory.

Unlike a regular persistent mount, AutoFS can automatically unmount the share after a period of inactivity.

#### Install AutoFS

```bash
dnf install -y autofs nfs-utils
```

#### Configure the master map

Created a dedicated AutoFS configuration file:

```bash
vim /etc/auto.master.d/nfs.autofs
```

Added:

```text
/- /etc/auto.nfs
```

`/-` specifies a direct mapping.

#### Configure NFS direct map

Created the map file:

```bash
vim /etc/auto.nfs
```

Added:

```text
/mnt/nfs/shared -fstype=nfs,vers=4,rw 10.219.122.167:/srv/nfs/shared
```

#### Start and enable AutoFS

```bash
systemctl enable --now autofs
```

#### Verify AutoFS service

```bash
systemctl status autofs
```

#### Test automatic mounting

Accessed the configured directory:

```bash
ls -la /mnt/nfs/shared
```

Verified the NFS mount:

```bash
findmnt /mnt/nfs/shared

findmnt -t nfs,nfs4
```

Accessing the directory triggers AutoFS to mount the NFS share.

**Important:** The `/etc/fstab` entry and AutoFS mapping are alternative ways of managing the same mount point. Before using AutoFS, remove or comment out the corresponding `/etc/fstab` entry and unmount the existing manual mount. Do not configure both simultaneously for `/mnt/nfs/shared`.

## Important NFS Commands

| Command                       | Purpose                                       |
| ----------------------------- | --------------------------------------------- |
| `exportfs -v`                 | Displays active NFS exports                   |
| `exportfs -rav`               | Reloads NFS export configurations             |
| `mount -t nfs`                | Mounts an NFS share manually                  |
| `umount`                      | Unmounts a filesystem                         |
| `findmnt -t nfs,nfs4`         | Displays NFS mounts                           |
| `systemctl status nfs-server` | Checks NFS server status                      |
| `systemctl status autofs`     | Checks AutoFS status                          |
| `mount -a`                    | Mounts filesystems configured in `/etc/fstab` |

## Final Verification Checklist

- [ ] NFS utilities installed on server and client.
- [ ] NFS server service enabled and running.
- [ ] LVM-backed directory exported successfully.
- [ ] NFS export restricted to the intended client subnet.
- [ ] Firewall configured to allow NFS traffic.
- [ ] NFS share mounted successfully on the client.
- [ ] Client access and file sharing verified.
- [ ] Persistent mounting configuration tested.
- [ ] AutoFS direct mapping configured.
- [ ] Accessing the directory triggers automatic mounting.
- [ ] AutoFS enabled to start after reboot.

## Outcome

Configured NFS-based network file sharing between RHEL systems using an LVM-backed storage directory.

Demonstrated manual NFS mounting, persistent mounting through `/etc/fstab`, and on-demand mounting using AutoFS direct mapping.

## Evidence

See the [NFS screenshots](https://github.com/amrit0x/Hardened-RHEL-File-Server/tree/main/implementation/Images) directory for server configuration, export verification, client mounting, and AutoFS testing.
