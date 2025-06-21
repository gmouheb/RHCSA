# Stratis Storage in Linux

This document covers Stratis, a volume-managing filesystem for Linux.

## Introduction to Stratis

Stratis is a local storage management solution for Linux that aims to make advanced storage features more accessible. It provides:

- Easy-to-use storage management
- Thin provisioning by default
- Snapshots
- Pool-based storage management
- File system resizing
- Monitoring capabilities

Stratis combines the functionality of a volume manager and a filesystem, using technologies like:
- Device Mapper (DM)
- XFS filesystem
- D-Bus for API access

## Stratis Architecture

Stratis has a layered architecture:

1. **Block Devices**: Physical storage devices (HDDs, SSDs, etc.)
2. **Storage Pools**: Collections of block devices
3. **Filesystems**: Created from storage pools
4. **Snapshots**: Point-in-time copies of filesystems

## Installing Stratis

```bash
# Install Stratis packages
dnf install stratis-cli stratisd

# Start and enable the Stratis daemon
systemctl enable --now stratisd
```

## Creating and Managing Stratis Storage

### Creating a Stratis Pool

```bash
# Create a pool named "mypool" using one or more block devices
stratis pool create mypool /dev/sdb /dev/sdc

# List available pools
stratis pool list
```

### Adding Devices to a Pool

```bash
# Add a device to an existing pool
stratis pool add-data mypool /dev/sdd
```

### Creating Filesystems

```bash
# Create a filesystem named "fs1" in the pool "mypool"
stratis filesystem create mypool fs1

# List filesystems
stratis filesystem list
```

### Mounting Stratis Filesystems

```bash
# Create a mount point
mkdir -p /mnt/stratis/fs1

# Mount the filesystem
mount /dev/stratis/mypool/fs1 /mnt/stratis/fs1

# Add to /etc/fstab for persistent mounting
echo "/dev/stratis/mypool/fs1 /mnt/stratis/fs1 xfs defaults,x-systemd.requires=stratisd.service 0 0" >> /etc/fstab
```

Note: The `x-systemd.requires=stratisd.service` option ensures that the Stratis daemon is started before mounting the filesystem.

## Working with Stratis Snapshots

Snapshots in Stratis are read-write by default and can be used as regular filesystems.

### Creating Snapshots

```bash
# Create a snapshot named "fs1-snap" of the filesystem "fs1"
stratis filesystem snapshot mypool fs1 fs1-snap

# List snapshots (shown as filesystems)
stratis filesystem list
```

### Using Snapshots

```bash
# Mount a snapshot
mkdir -p /mnt/stratis/fs1-snap
mount /dev/stratis/mypool/fs1-snap /mnt/stratis/fs1-snap

# Use the snapshot like any other filesystem
# You can read, write, or restore data from it
```

### Destroying Snapshots

```bash
# Unmount the snapshot first
umount /mnt/stratis/fs1-snap

# Destroy the snapshot
stratis filesystem destroy mypool fs1-snap
```

## Expanding Storage

### Adding Devices to a Pool

```bash
# Add more storage to a pool
stratis pool add-data mypool /dev/sde
```

Stratis automatically expands the pool capacity when new devices are added.

### Filesystem Expansion

Stratis filesystems automatically use the available space in the pool. There's no need to manually resize filesystems when the pool grows.

## Monitoring Stratis

### Checking Pool Status

```bash
# View detailed pool information
stratis pool list

# Check the amount of used and total space
stratis pool list --properties total_physical_size,total_physical_used
```

### Checking Filesystem Status

```bash
# View detailed filesystem information
stratis filesystem list

# Check the amount of used and total space
stratis filesystem list --properties used_size,total_size
```

## Destroying Stratis Resources

### Destroying Filesystems

```bash
# Unmount the filesystem first
umount /mnt/stratis/fs1

# Destroy the filesystem
stratis filesystem destroy mypool fs1
```

### Destroying Pools

```bash
# Destroy a pool (all filesystems in the pool must be destroyed first)
stratis pool destroy mypool
```

## Cache Tier Management

Stratis supports cache tiering, allowing you to use fast devices (like SSDs) as a cache for slower devices.

```bash
# Add a cache device to a pool
stratis pool add-cache mypool /dev/nvme0n1
```

## Encryption in Stratis

Stratis supports encryption using LUKS (Linux Unified Key Setup).

### Setting Up the Keyring

```bash
# Set up a key in the kernel keyring
stratis key set --capture-key mykey
# Enter your passphrase when prompted
```

### Creating an Encrypted Pool

```bash
# Create an encrypted pool
stratis pool create --key-desc mykey myencryptedpool /dev/sdb
```

### Creating Filesystems on Encrypted Pools

```bash
# Create a filesystem on the encrypted pool
stratis filesystem create myencryptedpool fs1
```

## Stratis vs. Traditional Storage Solutions

### Advantages of Stratis

1. **Ease of use**: Simplified management compared to LVM+filesystem
2. **Thin provisioning**: Efficient space utilization
3. **Snapshots**: Easy point-in-time copies
4. **Automatic filesystem resizing**: No manual intervention needed
5. **Modern design**: Built on proven technologies with a modern interface

### Limitations of Stratis

1. **Maturity**: Newer than established solutions like LVM
2. **Recovery options**: Limited compared to traditional filesystems
3. **Performance overhead**: Due to thin provisioning and layered design
4. **RAID support**: Limited compared to mdadm or hardware RAID

## Best Practices for Stratis

1. **Regular backups**: Despite snapshots, maintain regular backups
2. **Monitoring**: Regularly check pool usage to avoid running out of space
3. **Encryption planning**: Plan key management if using encryption
4. **Documentation**: Document your Stratis configuration
5. **Updates**: Keep Stratis packages updated for bug fixes and new features
6. **Testing**: Test recovery procedures before relying on them in production
7. **Redundancy**: Consider using multiple devices in pools for redundancy

## Troubleshooting Stratis

### Common Issues and Solutions

1. **Filesystem doesn't mount at boot:**
   - Ensure the correct mount options in `/etc/fstab`
   - Verify that `stratisd.service` is enabled

2. **Pool shows as unavailable:**
   ```bash
   # Restart the Stratis daemon
   systemctl restart stratisd
   
   # Check if devices are accessible
   lsblk
   ```

3. **Cannot destroy a filesystem:**
   - Ensure the filesystem is unmounted
   - Check if any snapshots of the filesystem exist

4. **Pool space not reclaimed after deleting files:**
   - Stratis uses thin provisioning and may not immediately reclaim space
   - Use `fstrim` to reclaim space:
     ```bash
     fstrim /mnt/stratis/fs1
     ```

## Stratis Command Reference

```bash
# List all Stratis commands
stratis --help

# Get help for a specific command
stratis pool --help

# List all pools
stratis pool list

# List all filesystems
stratis filesystem list

# List all known block devices
stratis blockdev list

# List daemon information
stratis daemon version
```