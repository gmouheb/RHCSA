# Logical Volume Management in Linux

This document covers Logical Volume Management (LVM) concepts and commands in Linux.

## Introduction to LVM

Logical Volume Management (LVM) provides a layer of abstraction between physical storage devices and file systems, offering benefits such as:

- Flexible storage management
- Online storage resizing
- Storage pooling across multiple disks
- Snapshots for backups
- Striping and mirroring capabilities

## LVM Architecture

LVM consists of three main components:

1. **Physical Volumes (PVs)**: Physical storage devices or partitions
2. **Volume Groups (VGs)**: Groups of physical volumes that form a storage pool
3. **Logical Volumes (LVs)**: Virtual partitions created from volume groups

## Setting Up LVM

### Step 1: Prepare Physical Volumes

```bash
# Install LVM tools if not already installed
dnf install lvm2

# Create partitions with Linux LVM type (8e) if needed
fdisk /dev/sdb
# Create a new partition and set type to Linux LVM (8e)

# Initialize physical volumes
pvcreate /dev/sdb1 /dev/sdc1

# View physical volume information
pvs
pvdisplay
pvdisplay /dev/sdb1
```

### Step 2: Create Volume Groups

```bash
# Create a volume group named "vg_data" from physical volumes
vgcreate vg_data /dev/sdb1 /dev/sdc1

# View volume group information
vgs
vgdisplay
vgdisplay vg_data
```

### Step 3: Create Logical Volumes

```bash
# Create a logical volume named "lv_data" with 10GB size
lvcreate -n lv_data -L 10G vg_data

# Create a logical volume using percentage of free space
lvcreate -n lv_backup -l 100%FREE vg_data

# View logical volume information
lvs
lvdisplay
lvdisplay /dev/vg_data/lv_data
```

### Step 4: Create File Systems and Mount

```bash
# Create a file system on the logical volume
mkfs.ext4 /dev/vg_data/lv_data
# or
mkfs.xfs /dev/vg_data/lv_data

# Create a mount point
mkdir -p /mnt/data

# Mount the logical volume
mount /dev/vg_data/lv_data /mnt/data

# Add to /etc/fstab for persistent mounting
echo "/dev/vg_data/lv_data /mnt/data ext4 defaults 0 2" >> /etc/fstab
# or using device mapper path
echo "/dev/mapper/vg_data-lv_data /mnt/data ext4 defaults 0 2" >> /etc/fstab
```

## Managing Physical Volumes

```bash
# Add a new physical volume
pvcreate /dev/sdd1

# Resize a physical volume (if underlying partition was resized)
pvresize /dev/sdb1

# Move data from one PV to another
pvmove /dev/sdb1 /dev/sdd1

# Remove a physical volume (must not be in use)
pvremove /dev/sdb1
```

## Managing Volume Groups

```bash
# Add physical volumes to a volume group
vgextend vg_data /dev/sdd1

# Remove physical volumes from a volume group
vgreduce vg_data /dev/sdb1

# Rename a volume group
vgrename vg_data vg_newdata

# Activate/deactivate a volume group
vgchange -a y vg_data  # Activate
vgchange -a n vg_data  # Deactivate

# Remove a volume group
vgremove vg_data
```

## Managing Logical Volumes

### Resizing Logical Volumes

```bash
# Extend a logical volume
lvextend -L +5G /dev/vg_data/lv_data  # Add 5GB
lvextend -L 15G /dev/vg_data/lv_data  # Set to 15GB
lvextend -l +100%FREE /dev/vg_data/lv_data  # Use all free space

# Resize the file system after extending the LV
# For ext4:
resize2fs /dev/vg_data/lv_data
# For XFS (only growing, not shrinking):
xfs_growfs /dev/vg_data/lv_data

# Extend LV and resize filesystem in one command
lvextend -r -L +5G /dev/vg_data/lv_data

# Reduce a logical volume (ext4 only, not XFS)
# First, unmount and check the filesystem
umount /mnt/data
e2fsck -f /dev/vg_data/lv_data
# Resize the filesystem first
resize2fs /dev/vg_data/lv_data 5G
# Then reduce the logical volume
lvreduce -L 5G /dev/vg_data/lv_data
```

### Removing Logical Volumes

```bash
# Unmount the logical volume
umount /mnt/data

# Remove the logical volume
lvremove /dev/vg_data/lv_data
```

### Renaming Logical Volumes

```bash
# Rename a logical volume
lvrename vg_data lv_data lv_newdata
```

## LVM Snapshots

Snapshots provide point-in-time copies of logical volumes, useful for backups.

### Creating Snapshots

```bash
# Create a snapshot with 1GB size
lvcreate -s -n lv_data_snapshot -L 1G /dev/vg_data/lv_data

# View snapshot information
lvs -o +devices,lv_size,lv_attr
```

### Using Snapshots for Backup

```bash
# Mount the snapshot
mkdir -p /mnt/snapshot
mount -o ro /dev/vg_data/lv_data_snapshot /mnt/snapshot

# Backup from the snapshot
tar -czf /backup/data_backup.tar.gz -C /mnt/snapshot .

# Unmount the snapshot
umount /mnt/snapshot
```

### Merging Snapshots Back

```bash
# Unmount the original volume
umount /mnt/data

# Merge the snapshot back to the original
lvconvert --merge /dev/vg_data/lv_data_snapshot
```

### Removing Snapshots

```bash
# Remove a snapshot
lvremove /dev/vg_data/lv_data_snapshot
```

## Advanced LVM Features

### Striped Logical Volumes

Striping distributes data across multiple physical volumes for improved performance.

```bash
# Create a striped logical volume across 2 PVs
lvcreate -n lv_striped -L 10G -i 2 -I 64 vg_data
# -i 2: Use 2 stripes
# -I 64: Use 64KB stripe size
```

### Mirrored Logical Volumes

Mirroring duplicates data across multiple physical volumes for redundancy.

```bash
# Create a mirrored logical volume
lvcreate -n lv_mirror -L 10G -m 1 vg_data
# -m 1: Create 1 mirror (2 copies total)
```

### Thin Provisioning

Thin provisioning allows over-allocation of storage.

```bash
# Create a thin pool
lvcreate -n thin_pool -L 20G vg_data --thinpool

# Create thin volumes from the pool
lvcreate -n thin_vol1 -V 10G --thin vg_data/thin_pool
lvcreate -n thin_vol2 -V 10G --thin vg_data/thin_pool
```

## LVM Backup and Restore

LVM automatically creates metadata backups in `/etc/lvm/backup/`.

```bash
# Manually backup LVM metadata
vgcfgbackup vg_data

# Restore LVM metadata
vgcfgrestore vg_data
```

## Troubleshooting LVM

### Common Issues and Solutions

1. **Missing Volume Groups after Reboot:**
   ```bash
   # Scan for all volume groups
   vgscan
   
   # Activate all volume groups
   vgchange -a y
   ```

2. **Corrupted Physical Volume:**
   ```bash
   # Check and repair PV metadata
   pvck /dev/sdb1
   ```

3. **Recovering Deleted Volume Groups:**
   ```bash
   # Restore from backup
   vgcfgrestore vg_data
   ```

4. **Insufficient Space for Snapshot:**
   ```bash
   # Extend snapshot size
   lvextend -L +1G /dev/vg_data/lv_data_snapshot
   ```

## LVM Best Practices

1. Always leave some free space in volume groups for snapshots and emergencies
2. Use meaningful names for volume groups and logical volumes
3. Document your LVM configuration
4. Regularly backup LVM metadata
5. Test snapshot and restore procedures before relying on them
6. Use UUIDs in `/etc/fstab` for logical volumes
7. Consider using thin provisioning for flexible space allocation
8. Monitor free space in volume groups and thin pools
9. Use the `-r` option with `lvextend` to resize the filesystem automatically
10. Be cautious with reducing logical volumes - always backup data first