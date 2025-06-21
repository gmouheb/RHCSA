# Partitions and Mounts in Linux

This document covers disk partitioning and mounting file systems in Linux.

## Understanding Storage Devices in Linux

Linux represents storage devices as files in the `/dev` directory:

- SATA/SCSI/USB drives: `/dev/sda`, `/dev/sdb`, etc.
- NVMe drives: `/dev/nvme0n1`, `/dev/nvme1n1`, etc.
- Virtual drives: `/dev/vda`, `/dev/vdb`, etc.
- Partitions: `/dev/sda1`, `/dev/sda2`, `/dev/nvme0n1p1`, etc.

## Disk Partitioning Schemes

### MBR (Master Boot Record)

- Traditional partitioning scheme
- Limited to 2TB disk size
- Maximum of 4 primary partitions
- Extended partitions allow for more logical partitions
- Uses `fdisk` for management

### GPT (GUID Partition Table)

- Modern partitioning scheme
- Supports disks larger than 2TB
- Supports up to 128 partitions by default
- More robust than MBR
- Uses `gdisk` or `parted` for management

## Viewing Disk and Partition Information

```bash
# List block devices
lsblk

# Show detailed information about block devices
lsblk -f

# Display disk partitions
fdisk -l

# Show disk usage
df -h

# Show disk usage by file system type
df -T

# Show detailed file system information
blkid
```

## Partitioning Tools

### fdisk (for MBR partitions)

```bash
# Start fdisk for a specific disk
fdisk /dev/sda

# Common fdisk commands:
# m - display help
# p - print partition table
# n - create new partition
# d - delete partition
# t - change partition type
# w - write changes and exit
# q - quit without saving changes
```

### gdisk (for GPT partitions)

```bash
# Start gdisk for a specific disk
gdisk /dev/sda

# Commands are similar to fdisk
```

### parted (for both MBR and GPT)

```bash
# Start parted for a specific disk
parted /dev/sda

# Common parted commands:
# print - display partition table
# mklabel msdos|gpt - create a new partition table
# mkpart - create a new partition
# rm - remove a partition
# quit - exit parted
```

## Creating Partitions - Step by Step

### Using fdisk (MBR)

```bash
# Start fdisk
fdisk /dev/sda

# Create a new primary partition
Command: n
Partition type: p
Partition number: 1
First sector: [press Enter for default]
Last sector: +10G (for a 10GB partition)

# Change partition type (optional)
Command: t
Partition number: 1
Hex code: 83 (Linux)

# Write changes
Command: w
```

### Using parted (GPT)

```bash
# Start parted
parted /dev/sda

# Create a GPT partition table if needed
mklabel gpt

# Create a new partition
mkpart primary ext4 0% 10GB

# Set the name (optional)
name 1 data

# Verify the partition
print

# Exit parted
quit
```

## File Systems in Linux

Common file systems in Linux:

- **ext4**: Default for many Linux distributions
- **xfs**: High-performance file system, default in RHEL/CentOS
- **btrfs**: Modern file system with advanced features
- **vfat/fat32**: Compatible with Windows
- **ntfs**: Windows file system (read/write support via ntfs-3g)
- **swap**: Swap space for virtual memory

## Creating File Systems

```bash
# Create an ext4 file system
mkfs.ext4 /dev/sda1

# Create an XFS file system
mkfs.xfs /dev/sda2

# Create a Btrfs file system
mkfs.btrfs /dev/sda3

# Create a FAT32 file system
mkfs.vfat -F 32 /dev/sda4

# Create a swap area
mkswap /dev/sda5
```

## Mounting File Systems

### Manual Mounting

```bash
# Create a mount point
mkdir -p /mnt/data

# Mount a file system
mount /dev/sda1 /mnt/data

# Mount with specific options
mount -o rw,noexec,nosuid /dev/sda1 /mnt/data

# Unmount a file system
umount /mnt/data
# or
umount /dev/sda1
```

### Persistent Mounting with /etc/fstab

The `/etc/fstab` file controls automatic mounting at boot time.

Format:
```
<device> <mount_point> <file_system_type> <options> <dump> <fsck_order>
```

Example entries:
```
# Mount by device
/dev/sda1    /mnt/data    ext4    defaults    0    2

# Mount by UUID (recommended)
UUID=1234-5678-90ab-cdef    /mnt/data    ext4    defaults    0    2

# Mount by label
LABEL=data    /mnt/data    ext4    defaults    0    2

# Mount a network share
//server/share    /mnt/network    cifs    credentials=/etc/cifs-credentials,uid=1000    0    0

# Mount swap
/dev/sda5    none    swap    sw    0    0
```

### Testing fstab Entries

```bash
# Test mounting all entries in fstab
mount -a

# Test mounting a specific entry
mount /mnt/data
```

## Common Mount Options

- **defaults**: rw, suid, dev, exec, auto, nouser, async
- **ro/rw**: Read-only or read-write
- **noexec/exec**: Prevent/allow execution of binaries
- **nosuid/suid**: Disable/enable SUID and SGID bits
- **nodev/dev**: Disable/enable device files
- **noauto/auto**: Don't mount/mount automatically at boot
- **user/nouser**: Allow/disallow regular users to mount
- **async/sync**: Asynchronous/synchronous I/O
- **noatime**: Don't update access times (improves performance)

## Swap Space

### Creating and Managing Swap

```bash
# Create a swap partition
mkswap /dev/sda5

# Activate swap
swapon /dev/sda5

# Deactivate swap
swapoff /dev/sda5

# Show swap usage
swapon --show
# or
free -h
```

### Creating a Swap File

```bash
# Create a 1GB file
dd if=/dev/zero of=/swapfile bs=1M count=1024

# Set permissions
chmod 600 /swapfile

# Format as swap
mkswap /swapfile

# Activate swap
swapon /swapfile

# Add to fstab for persistence
echo "/swapfile    none    swap    sw    0    0" >> /etc/fstab
```

## Working with Labels and UUIDs

```bash
# View UUIDs and labels
blkid

# Set a label for ext4
e2label /dev/sda1 data

# Set a label for XFS
xfs_admin -L data /dev/sda2

# Find UUID of a device
lsblk -f
# or
blkid /dev/sda1
```

## Checking and Repairing File Systems

```bash
# Check an ext4 file system
fsck.ext4 /dev/sda1
# or
e2fsck /dev/sda1

# Check an XFS file system
xfs_repair /dev/sda2

# Force check on next boot (ext4)
tune2fs -c 1 /dev/sda1
```

## Mounting Special File Systems

```bash
# Mount a tmpfs (RAM) file system
mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk

# Add to fstab
tmpfs    /mnt/ramdisk    tmpfs    size=1G    0    0

# Mount proc file system
mount -t proc proc /proc

# Mount sysfs
mount -t sysfs sys /sys
```

## Automounting with Autofs

```bash
# Install autofs
dnf install autofs

# Configure master map file (/etc/auto.master)
/mnt/auto    /etc/auto.misc    --timeout=60

# Configure map file (/etc/auto.misc)
data    -fstype=ext4    :/dev/sda1

# Start and enable autofs
systemctl enable --now autofs
```

## Troubleshooting Mount Issues

### Common Issues and Solutions

1. **"Device is busy" when unmounting:**
   ```bash
   # Find processes using the mount
   fuser -m /mnt/data
   
   # Kill processes using the mount
   fuser -km /mnt/data
   ```

2. **File system errors:**
   ```bash
   # Unmount and check the file system
   umount /dev/sda1
   fsck -f /dev/sda1
   ```

3. **Wrong UUID in fstab:**
   ```bash
   # Boot in rescue mode and fix /etc/fstab
   # Or use the following to temporarily skip mounting
   mount -o remount,rw /
   ```

4. **Missing mount point:**
   ```bash
   # Create the mount point
   mkdir -p /mnt/data
   ```

## Best Practices

1. Use UUIDs instead of device names in `/etc/fstab`
2. Create separate partitions for `/home`, `/var`, and `/tmp`
3. Use appropriate file systems for different use cases
4. Regularly check file system health
5. Consider performance options like `noatime` for busy systems
6. Use logical volume management (LVM) for flexibility
7. Document your partition scheme and mount options
8. Test mount configurations before rebooting
9. Keep backups of critical data and `/etc/fstab`
10. Use appropriate permissions for mount points