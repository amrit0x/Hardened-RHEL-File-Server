# 1. LVM Storage Configuration

## Objective

Configure Logical Volume Manager (LVM) on RHEL to provide flexible storage for the network file server.

The storage will use an XFS filesystem mounted at `/srv/nfs/shared`.

## Storage Architecture

```text
/dev/sdb (10 GB)
     |
     v
Physical Volume (PV)
     |
     v
Volume Group: vg_nfs
     |
     v
Logical Volume: lv_nfs
     |
     v
XFS Filesystem
     |
     v
/srv/nfs/shared
```

**Storage allocation:**

- Disk: 10 GB
- Initial Logical Volume: 6 GB
- Additional space reserved in VG: approximately 4 GB
- Final Logical Volume after extension: 7 GB
- Remaining VG space: approximately 3 GB

The initial free space allows online LVM expansion to be demonstrated.

## Implementation

### 1. Verify Storage Device

Identified the additional 10 GB disk and checked for existing filesystem signatures and LVM configurations.

**Commands:**

```bash
lsblk -f /dev/sdb
wipefs -n /dev/sdb
pvs
vgs
lvs
```

**Purpose:**
- Identify the available disk.
- Check for existing filesystem signatures.
- Review existing physical volumes, volume groups and logical volumes.
- Ensure existing storage configurations remain untouched.

**Safety:** Only proceed with disk initialization after confirming that `/dev/sdb` is the correct dedicated disk and contains no data that needs to be preserved.

### 2. Create Physical Volume (PV)

Initialized `/dev/sdb` for LVM management.

```bash
pvcreate /dev/sdb
```

**Verification:**

```bash
pvs
```

### 3. Create Volume Group (VG)

Created a volume group named `vg_nfs` using the physical volume.

```bash
vgcreate vg_nfs /dev/sdb
```

**Verification:**

```bash
vgs
```

### 4. Create Logical Volume (LV)

Created an initial 6 GB logical volume named `lv_nfs`.

```bash
lvcreate -L 6G -n lv_nfs vg_nfs
```

**Verification:**

```bash
lvs
vgs -o vg_name,vg_size,vg_free
```

Approximately 4 GB was reserved for future storage expansion.

### 5. Create XFS Filesystem

Created an XFS filesystem on the logical volume.

```bash
mkfs.xfs /dev/vg_nfs/lv_nfs
```

**Verification:**

```bash
blkid /dev/vg_nfs/lv_nfs
```

### 6. Mount the Filesystem

Created the shared storage directory and mounted the logical volume.

```bash
mkdir -p /srv/nfs/shared

mount /dev/vg_nfs/lv_nfs /srv/nfs/shared
```

**Verification:**

```bash
df -hT /srv/nfs/shared

findmnt /srv/nfs/shared
```

### 7. Configure Persistent Mounting

Configured `/etc/fstab` to automatically mount the filesystem after a server reboot.

Retrieved the filesystem UUID:

```bash
blkid /dev/vg_nfs/lv_nfs
```

Added the following entry to `/etc/fstab`, replacing the placeholder with the actual UUID:

```text
UUID=<ACTUAL-UUID> /srv/nfs/shared xfs defaults 0 0
```

Validated and tested the configuration:

```bash
findmnt --verify --verbose

umount /srv/nfs/shared

mount -a

findmnt /srv/nfs/shared

df -hT /srv/nfs/shared
```

### 8. Extend Logical Volume Online

Demonstrated online storage expansion by increasing the logical volume from 6 GB to 7 GB.

Extended the logical volume by 1 GB:

```bash
lvextend -L +1G /dev/vg_nfs/lv_nfs
```

Expanded the mounted XFS filesystem:

```bash
xfs_growfs /srv/nfs/shared
```
if we was used this command: ```lvextend -r -L +1G /dev/vg_nfs/lv_nfs``` then we does not need to run this separately ```xfs_growfs /srv/nfs/shared ``` because ```-r``` do it by it self

**Verification:**

```bash
lvs

vgs

df -hT /srv/nfs/shared
```

**Important concepts:**

| Command | Purpose |
|---|---|
| `pvcreate` | Initializes a physical volume |
| `vgcreate` | Creates a volume group |
| `lvcreate` | Creates a logical volume |
| `lvextend` | Extends logical volume capacity |
| `mkfs.xfs` | Creates an XFS filesystem |
| `xfs_growfs` | Expands a mounted XFS filesystem |

## Final Verification

Executed the following commands to verify the completed LVM configuration:

```bash
pvs

vgs

lvs -o lv_name,vg_name,lv_size,devices

lsblk -f

df -hT

findmnt /srv/nfs/shared
```

## Final Configuration

| Component | Configuration |
|---|---|
| Disk | /dev/sdb (10 GB) |
| Physical Volume | /dev/sdb |
| Volume Group | vg_nfs |
| Logical Volume | lv_nfs (7 GB) |
| Filesystem | XFS |
| Mount Point | /srv/nfs/shared |
| Persistent Mount | Configured |

*Storage values are approximate and should be updated to match actual command outputs.*

## Outcome

Successfully configured LVM-based storage on RHEL, created an XFS filesystem, configured persistent mounting, and demonstrated logical volume and XFS filesystem expansion.

This storage will serve as the foundation for the NFS network file-sharing configuration.

## Evidence

See the [LVM screenshots](../screenshots/lvm/) directory for terminal outputs and configuration verification.
