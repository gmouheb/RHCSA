# Monitoring System Activity in Linux

This document covers tools and techniques for monitoring system activity in Linux.

## System Load and Resource Usage

### Using `top` and `htop`

```bash
# Basic system monitoring with top
top

# Enhanced monitoring with htop (may need to be installed)
htop
```

Key metrics in top/htop:
- Load average
- CPU usage
- Memory usage
- Running processes

### System Activity Reporter (SAR)

```bash
# Install SAR if not already installed
dnf install sysstat

# Enable and start the service
systemctl enable sysstat
systemctl start sysstat

# View current CPU statistics
sar -u

# View memory usage
sar -r

# View disk I/O statistics
sar -d

# View network statistics
sar -n DEV
```

## Process Monitoring

### Using `ps` Command

```bash
# List all processes
ps aux

# List processes in a hierarchical format
ps -ef --forest

# List processes by a specific user
ps -u username

# List processes sorted by CPU usage
ps aux --sort=-%cpu

# List processes sorted by memory usage
ps aux --sort=-%mem
```

### Process Monitoring with `pgrep` and `pkill`

```bash
# Find process IDs by name
pgrep process_name

# Kill processes by name
pkill process_name

# Send a specific signal to a process
pkill -SIGTERM process_name
```

## Memory Usage

### Using `free` Command

```bash
# Display memory usage in human-readable format
free -h

# Display memory usage in megabytes
free -m
```

### Using `vmstat` Command

```bash
# Display virtual memory statistics
vmstat

# Display memory statistics every 2 seconds for 5 times
vmstat 2 5
```

## Disk Usage and I/O

### Disk Space Usage

```bash
# Display disk space usage
df -h

# Display disk usage for specific directory
du -sh /path/to/directory

# Find largest directories
du -h /path | sort -hr | head -10
```

### Disk I/O Monitoring

```bash
# Monitor disk I/O in real-time
iostat -x 2

# Monitor I/O for specific device
iostat -x 2 /dev/sda
```

## Network Monitoring

### Using `netstat` and `ss`

```bash
# List all listening ports
ss -tuln

# List all established connections
ss -tua

# List all TCP connections
netstat -ant

# List all UDP connections
netstat -anu
```

### Using `iftop` and `nethogs`

```bash
# Monitor network bandwidth usage by interface
iftop -i eth0

# Monitor network bandwidth usage by process
nethogs eth0
```

## Log Monitoring

### System Logs

```bash
# View system logs with journalctl
journalctl

# Follow system logs in real-time
journalctl -f

# View logs for a specific service
journalctl -u service_name
```

### Traditional Log Files

Important log files to monitor:
- `/var/log/messages` - General system messages
- `/var/log/secure` - Authentication and security logs
- `/var/log/audit/audit.log` - Audit logs
- `/var/log/httpd/` - Web server logs

## Performance Monitoring Tools

### Performance Co-Pilot (PCP)

```bash
# Install PCP
dnf install pcp

# Start PCP services
systemctl start pmcd pmlogger

# Use PCP tools
pminfo
pmstat
```

### Tuned Profiles

```bash
# Install tuned
dnf install tuned

# List available profiles
tuned-adm list

# Set a specific profile
tuned-adm profile profile_name

# Show current active profile
tuned-adm active
```