# NFS and Autofs in Linux

This document covers Network File System (NFS) and Autofs configuration and management in Linux systems.

## Introduction to NFS

Network File System (NFS) is a distributed file system protocol that allows a user on a client computer to access files over a network as if those files were on the local system. NFS provides:

- Transparent access to remote files
- Centralized storage management
- Shared access to common data
- Reduced local storage requirements

## NFS Versions

- **NFSv3**: Older version, still widely used
- **NFSv4**: Current version with improved security, performance, and features
- **NFSv4.1/4.2**: Latest versions with additional features

## Setting Up an NFS Server

### Installing NFS Server Packages

```bash
# On RHEL/CentOS/Fedora
dnf install nfs-utils

# On Debian/Ubuntu
apt install nfs-kernel-server
```

### Configuring NFS Exports

The main configuration file for NFS exports is `/etc/exports`. Each line defines a shared directory and its access permissions.

```bash
# Edit the exports file
vi /etc/exports
```

Example `/etc/exports` entries:
```
# Basic export to a single host
/shared/data    192.168.1.10(rw,sync)

# Export to a network with read-only access
/shared/docs    192.168.1.0/24(ro,sync)

# Export with multiple options
/shared/projects    *.example.com(rw,sync,no_root_squash)

# Export to everyone (not recommended for security reasons)
/shared/public    *(ro)
```

### Common Export Options

- **rw**: Read and write access
- **ro**: Read-only access
- **sync**: Synchronous writes (more reliable but slower)
- **async**: Asynchronous writes (faster but less reliable)
- **no_root_squash**: Remote root user has root privileges
- **root_squash**: Remote root user is mapped to anonymous user (default)
- **all_squash**: All remote users are mapped to anonymous user
- **anonuid=UID**: Set the UID of the anonymous user
- **anongid=GID**: Set the GID of the anonymous user
- **secure**: Require connections from ports below 1024 (default)
- **insecure**: Allow connections from ports above 1024

### Applying Export Configuration

```bash
# Export all directories listed in /etc/exports
exportfs -a

# Re-export all directories (refresh)
exportfs -r

# Unexport all directories
exportfs -u

# View current exports
exportfs -v
```

### Starting and Enabling NFS Services

```bash
# Start and enable NFS server
systemctl enable --now nfs-server

# Check NFS server status
systemctl status nfs-server
```

### Firewall Configuration for NFS Server

```bash
# Allow NFS through firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

## Setting Up an NFS Client

### Installing NFS Client Packages

```bash
# On RHEL/CentOS/Fedora
dnf install nfs-utils

# On Debian/Ubuntu
apt install nfs-common
```

### Viewing Available NFS Exports

```bash
# Show available exports on a server
showmount -e server_ip
```

### Mounting NFS Shares Manually

```bash
# Create a mount point
mkdir -p /mnt/nfs

# Mount an NFS share
mount -t nfs server_ip:/shared/data /mnt/nfs

# Mount with specific options
mount -t nfs -o rw,soft,timeo=30 server_ip:/shared/data /mnt/nfs
```

### Common Mount Options

- **rw/ro**: Read-write or read-only access
- **soft/hard**: Soft or hard mount (soft returns errors, hard keeps retrying)
- **timeo=n**: Timeout value in tenths of a second
- **retrans=n**: Number of retransmissions before error
- **rsize=n**: Read buffer size
- **wsize=n**: Write buffer size
- **noatime**: Don't update access times
- **nodiratime**: Don't update directory access times
- **vers=n**: NFS protocol version (3 or 4)

### Persistent NFS Mounts with /etc/fstab

Add entries to `/etc/fstab` for automatic mounting at boot:

```
# Format: server:/export  mount_point  nfs  options  0 0
server_ip:/shared/data  /mnt/nfs  nfs  rw,soft,timeo=30  0 0
```

```bash
# Test fstab entry
mount -a
```

### Firewall Configuration for NFS Client

```bash
# Allow NFS client through firewall
firewall-cmd --permanent --add-service=nfs-client
firewall-cmd --reload
```

## NFS Security

### Using NFSv4 with Kerberos

NFSv4 can use Kerberos for authentication, providing stronger security:

```bash
# Server export with Kerberos
/shared/secure    *.example.com(rw,sync,sec=krb5p)
```

```bash
# Client mount with Kerberos
mount -t nfs4 -o sec=krb5p server_ip:/shared/secure /mnt/secure
```

Kerberos security levels:
- **krb5**: Authentication only
- **krb5i**: Authentication and integrity
- **krb5p**: Authentication, integrity, and privacy (encryption)

### NFS and SELinux

When using SELinux with NFS:

```bash
# Allow SELinux to access NFS shares
setsebool -P nfs_export_all_rw 1
setsebool -P nfs_export_all_ro 1

# For home directories
setsebool -P use_nfs_home_dirs 1
```

## Introduction to Autofs

Autofs is a service that automatically mounts filesystems when they are accessed and unmounts them after a period of inactivity. Benefits include:

- On-demand mounting
- Automatic unmounting of idle filesystems
- Reduced system resource usage
- Simplified client configuration

## Setting Up Autofs

### Installing Autofs

```bash
# On RHEL/CentOS/Fedora
dnf install autofs

# On Debian/Ubuntu
apt install autofs
```

### Configuring Autofs

Autofs uses two types of configuration files:
- Master map file (`/etc/auto.master`)
- Map files that define specific mounts

#### Master Map Configuration

Edit `/etc/auto.master`:
```
# Format: mount-point map-name [options]
/mnt/nfs    /etc/auto.nfs    --timeout=60
```

#### Map File Configuration

Create the map file referenced in the master map (e.g., `/etc/auto.nfs`):
```
# Format: mount-point [-options] server:/export
data    -rw,soft    server_ip:/shared/data
docs    -ro         server_ip:/shared/docs
```

This will create mounts at `/mnt/nfs/data` and `/mnt/nfs/docs` when accessed.

### Direct Maps

Direct maps specify the full path of each mount point:

In `/etc/auto.master`:
```
/-    /etc/auto.direct
```

In `/etc/auto.direct`:
```
/mnt/direct/data    -rw,soft    server_ip:/shared/data
/mnt/direct/docs    -ro         server_ip:/shared/docs
```

### Executable Maps

You can use executable scripts as map files:

In `/etc/auto.master`:
```
/mnt/dynamic    /etc/auto.script    --timeout=60
```

Create an executable script `/etc/auto.script`:
```bash
#!/bin/bash
# This is an example of an executable map

case $1 in
  data)
    echo "-rw,soft server_ip:/shared/data"
    ;;
  docs)
    echo "-ro server_ip:/shared/docs"
    ;;
esac
```

```bash
# Make the script executable
chmod +x /etc/auto.script
```

### Starting and Enabling Autofs

```bash
# Start and enable Autofs
systemctl enable --now autofs

# Check Autofs status
systemctl status autofs

# Reload Autofs configuration
systemctl reload autofs
```

## Advanced Autofs Features

### Wildcard Maps

Wildcard maps allow flexible mounting based on the requested directory:

```
# In /etc/auto.misc
*    -rw,soft    server_ip:/shared/&
```

This will mount `server_ip:/shared/dirname` at `/mnt/misc/dirname`.

### Using LDAP for Autofs Maps

Autofs can use LDAP to store mount information:

```
# In /etc/auto.master
/mnt/ldap    ldap:ou=automount,dc=example,dc=com
```

### Autofs Timeout and Browse Options

```
# In /etc/auto.master
/mnt/nfs    /etc/auto.nfs    --timeout=300 --ghost
```

- **timeout**: Time in seconds before unmounting (default: 300)
- **ghost**: Create empty directories for all map entries (for browsing)

## Troubleshooting NFS and Autofs

### Common NFS Issues

1. **Connection refused**:
   ```bash
   # Check if NFS server is running
   systemctl status nfs-server
   
   # Check if RPC services are running
   rpcinfo -p
   
   # Check firewall
   firewall-cmd --list-all
   ```

2. **Permission denied**:
   ```bash
   # Check export permissions
   cat /etc/exports
   
   # Check file permissions
   ls -la /shared/data
   
   # Check user mapping
   id
   ```

3. **Stale file handle**:
   ```bash
   # Unmount and remount the share
   umount /mnt/nfs
   mount -t nfs server_ip:/shared/data /mnt/nfs
   ```

### NFS Diagnostic Commands

```bash
# Check NFS server status
nfsstat -s

# Check NFS client status
nfsstat -c

# Check RPC services
rpcinfo -p server_ip

# Show NFS server statistics
nfsiostat

# Monitor NFS operations
nfswatch
```

### Common Autofs Issues

1. **Mounts not working**:
   ```bash
   # Check Autofs service
   systemctl status autofs
   
   # Check master map syntax
   cat /etc/auto.master
   
   # Check map file syntax
   cat /etc/auto.nfs
   
   # Increase logging
   systemctl stop autofs
   automount -f -v
   ```

2. **Timeout issues**:
   ```bash
   # Adjust timeout in /etc/auto.master
   /mnt/nfs    /etc/auto.nfs    --timeout=120
   
   # Reload configuration
   systemctl reload autofs
   ```

### Autofs Diagnostic Commands

```bash
# Show current Autofs mounts
automount -m

# Debug Autofs in foreground
systemctl stop autofs
automount -f -v

# Check Autofs logs
journalctl -u autofs
```

## Performance Tuning

### NFS Server Performance

```bash
# Increase NFS threads in /etc/nfs.conf
[nfsd]
threads=16

# Use async exports for better performance (with caution)
/shared/data    192.168.1.0/24(rw,async)

# Adjust rsize/wsize on server
echo "RPCNFSDARGS='-N 2 -N 3 -U -T'" > /etc/sysconfig/nfs
```

### NFS Client Performance

```bash
# Mount with optimized options
mount -t nfs -o rw,rsize=1048576,wsize=1048576,noatime server_ip:/shared/data /mnt/nfs

# Use NFSv4 for better performance
mount -t nfs4 server_ip:/shared/data /mnt/nfs

# Use TCP instead of UDP
mount -t nfs -o proto=tcp server_ip:/shared/data /mnt/nfs
```

### Autofs Performance

```bash
# Adjust timeout values
/mnt/nfs    /etc/auto.nfs    --timeout=600

# Use browse mode only when needed
/mnt/nfs    /etc/auto.nfs    --timeout=600 --nobrowse
```

## Best Practices

1. **Use NFSv4 when possible** for better security and performance
2. **Implement proper access controls** in exports
3. **Use specific IP addresses or networks** instead of wildcards
4. **Consider using Kerberos** for sensitive data
5. **Regularly back up NFS data**
6. **Monitor NFS performance** and adjust settings as needed
7. **Use Autofs for non-critical mounts** to improve system performance
8. **Document your NFS and Autofs configuration**
9. **Test failover scenarios** if using NFS for critical applications
10. **Keep NFS software updated** to address security vulnerabilities