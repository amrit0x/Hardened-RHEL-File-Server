# 4. SELinux Configuration

## Objective

Configure SELinux to protect the NFS shared directory
while keeping SELinux in Enforcing mode.

## Implementation
------------ Run on: RHEL Server -----------
### 1. Verify SELinux Status

getenforce
sestatus

### 2. Configure Persistent File Context


semanage fcontext -a -t public_content_rw_t \
'/srv/nfs/shared(/.*)?'

restorecon -Rv /srv/nfs/shared

### 3. Configure NFS SELinux Booleans


setsebool -P nfs_export_all_rw off
setsebool -P nfs_export_all_ro off
setsebool -P nfsd_anon_write on

### 4. Verify Configuration


ls -ldZ /srv/nfs/shared

getsebool nfs_export_all_ro \
nfs_export_all_rw nfsd_anon_write

### 5. Test NFS Access

(Run on: RHEL Client)

touch /mnt/nfs/shared/selinux-test.txt

(Run on: RHEL Server)

ls -lZ /srv/nfs/shared/selinux-test.txt

## Outcome

Configured SELinux file-context labeling for the NFS
shared directory and verified client-side file access.
