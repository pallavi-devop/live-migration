# Virtual Machine Migration Procedure

## Overview

This document outlines the steps for migrating a virtual machine (VM) from one host to another using the `virsh` command-line tool with SSH and NFS for storage. The destination host is specified, and the user is `ptsadmin`.

## Prerequisites

1. Ensure both source and destination hosts have:
   - QEMU/KVM installed.
   - Libvirt installed.
   - NFS client installed.

2. Proper SSH access between the two hosts.
3. NFS share properly configured on the storage server.

## Migration Steps

### 1. Verify VM Availability on Destination Host

Run the following command to check the existing VMs on the destination host:

```bash
virsh -c qemu+ssh://ptsadmin@<destination_host>/system list --all
```

### 2. Set Up SSH Key Authentication

If SSH key authentication is not set up, perform the following steps:

1. **Generate SSH Key Pair**:

   ```bash
   ssh-keygen
   ```

   Accept the default location and optionally set a passphrase.

2. **Copy the SSH Key to the Destination Host**:

   ```bash
   ssh-copy-id ptsadmin@<destination_host>
   ```

### 3. Verify SSH Access

Log into the destination host to ensure SSH is functioning:

```bash
ssh ptsadmin@<destination_host>
```

### 4. Check VM List Again

Re-run the command to list VMs:

```bash
virsh -c qemu+ssh://ptsadmin@<destination_host>/system list --all
```

### 5. Prepare for Migration

Verify that the NFS share is mounted on both the source and destination hosts:

```bash
df -hT
```

Check the contents of the `/vmdisks` directory:

```bash
ls /vmdisks/
```

### 6. Perform Live Migration

Execute the migration command:

```bash
virsh migrate --live test qemu+ssh://ptsadmin@<destination_host>/system
```

### 7. Verify the Migration

After migration, check the VM list again:

```bash
virsh -c qemu+ssh://ptsadmin@<destination_host>/system list --all
```

### 8. Handle Disk Images (if needed)

If the migration failed due to disk image issues, verify the images and move them as necessary:

```bash
mv test-clone.qcow2 /vmdisks/
mv test.qcow2 /vmdisks/
```

### 9. Check NFS Client Status

Ensure the NFS client is active:

```bash
systemctl status nfs-client.target
```

### 10. Mount the NFS Share

If not mounted, run the following commands:

```bash
mount <nfs_server>:/<nfs_share> /vmdisks/
```

### 11. Final Migration Attempt

Reattempt the migration after verifying disk paths:

```bash
virsh migrate --live test qemu+ssh://ptsadmin@<destination_host>/system
```

### 12. Optional Migration of Clone

If applicable, migrate the cloned VM as follows:

```bash
virsh migrate --live test-clone qemu+ssh://ptsadmin@<destination_host>/system
```

---

This document provides detailed steps for migrating a virtual machine, ensuring the migration process is carried out smoothly while taking into consideration SSH, NFS, and disk configurations.
