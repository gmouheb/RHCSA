# Lab 0: Disk Partitioning and Mounting - Solution

This document provides the solution for creating and managing disk partitions, formatting them with different filesystems, and mounting them persistently.

## Solution

### Requirement 1: Create a primary partition with Ext4
```bash
# Check current block devices
lsblk

# Create a primary partition
fdisk /dev/nvme0n2
# Enter the following commands in fdisk:
# n (new partition)
# p (primary)
# [Enter] (default partition number)
# [Enter] (default first sector)
# +1G (size)
# w (write changes)

# The previous commands will create a primary device with the name /dev/nvme0n2p1

# Create mount point
mkdir -p /mounts/lab0

# Format with ext4
mkfs.ext4 /dev/nvme0n2p1

# Get UUID
blkid

# Add to fstab for persistent mounting
echo "UUID=<your-uuid-here> /mounts/lab0 ext4 defaults 0 0" >> /etc/fstab
```

### Requirement 2: Create an extended partition with XFS
```bash
# Create an extended partition
fdisk /dev/nvme0n2
# Enter the following commands in fdisk:
# n (new partition)
# e (extended)
# [Enter] (default partition number)
# [Enter] (default first sector)
# [Enter] (default last sector - uses all remaining space)
# p (print partition table to verify)

# The previous commands will create an extended device with the name /dev/nvme0n2p2

# Create a logical partition within the extended partition
# n (new partition)
# [Enter] (default - logical partition)
# [Enter] (default first sector)
# +500M (size)
# w (write changes)

# The previous commands will create a device with the name /dev/nvme0n2p5 with the size 500M

# Create mount point
mkdir -p /mounts/xfslab0

# Format with XFS and add label
mkfs.xfs -L myxfslab0 /dev/nvme0n2p5

# Add to fstab for persistent mounting using label
echo "LABEL=myxfslab0 /mounts/xfslab0 xfs defaults 0 0" >> /etc/fstab
```

### Requirement 3: Create a swap partition
```bash
# Create another logical partition for swap
fdisk /dev/nvme0n2
# Enter the following commands in fdisk:
# n (new partition)
# [Enter] (default - logical partition)
# [Enter] (default first sector)
# +500M (size)
# t (change partition type)
# [Enter] (select the partition you just created)
# 82 or "swap" (swap partition type)
# p (print partition table to verify)
# w (write changes)

# The previous commands will create a swap partition with the name /dev/nvme0n2p6 with the size 500M

# Format as swap
mkswap /dev/nvme0n2p6

# Add to fstab for persistent mounting
echo "/dev/nvme0n2p6 none swap defaults 0 0" >> /etc/fstab

# Verify fstab entries
findmnt --verify
```
