# Tuning Performance in Linux

This document covers performance tuning concepts and techniques for Linux systems.

## Understanding System Performance

Performance tuning involves optimizing various system components:
- CPU usage
- Memory management
- Disk I/O
- Network performance
- Application performance

## Performance Monitoring Tools

### Basic Monitoring Tools

```bash
# System activity reporter
sar -u 1 10  # CPU usage every 1 second for 10 samples
sar -r 1 10  # Memory usage
sar -d 1 10  # Disk I/O
sar -n DEV 1 10  # Network statistics

# Process monitoring
top
htop
ps aux --sort=-%cpu  # Sort by CPU usage
ps aux --sort=-%mem  # Sort by memory usage

# Memory usage
free -h
vmstat 1 10

# Disk usage
df -h
du -sh /path/to/directory
iostat -x 1 10
```

### Advanced Monitoring Tools

```bash
# Install performance monitoring tools
dnf install perf sysstat dstat iotop

# Performance analysis with perf
perf stat command
perf record -a -g sleep 10
perf report

# I/O monitoring
iotop
```

## CPU Performance Tuning

### CPU Governors

```bash
# View available CPU governors
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# View current CPU governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set CPU governor (temporary)
echo "performance" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

### Process Priorities

```bash
# Run a process with higher priority
nice -n -10 command

# Change priority of a running process
renice -n -10 -p PID
```

## Memory Performance Tuning

### Swappiness

```bash
# View current swappiness value
cat /proc/sys/vm/swappiness

# Set swappiness temporarily
sysctl -w vm.swappiness=10

# Set swappiness permanently
echo "vm.swappiness=10" >> /etc/sysctl.conf
sysctl -p
```

### Transparent Huge Pages

```bash
# Check THP status
cat /sys/kernel/mm/transparent_hugepage/enabled

# Disable THP temporarily
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# Disable THP permanently (add to /etc/rc.local)
echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.local
chmod +x /etc/rc.local
```

## Disk I/O Performance Tuning

### I/O Schedulers

```bash
# View current I/O scheduler
cat /sys/block/sda/queue/scheduler

# Change I/O scheduler temporarily
echo "deadline" > /sys/block/sda/queue/scheduler

# Change I/O scheduler permanently
# Add to /etc/default/grub:
# GRUB_CMDLINE_LINUX="elevator=deadline"
# Then run: grub2-mkconfig -o /boot/grub2/grub.cfg
```

### File System Tuning

```bash
# Mount options for performance
# Add to /etc/fstab:
# /dev/sda1 /mount/point ext4 defaults,noatime,nodiratime 0 0

# Tune ext4 filesystem
tune2fs -o journal_data_writeback /dev/sda1
```

## Network Performance Tuning

### TCP Settings

```bash
# Increase TCP buffer sizes
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"

# Enable TCP BBR congestion control
sysctl -w net.core.default_qdisc=fq
sysctl -w net.ipv4.tcp_congestion_control=bbr
```

## Using Tuned Profiles

Tuned is a daemon that monitors and optimizes system performance.

```bash
# Install tuned
dnf install tuned

# Start and enable tuned
systemctl enable --now tuned

# List available profiles
tuned-adm list

# View current active profile
tuned-adm active

# Set a specific profile
tuned-adm profile throughput-performance  # For servers
tuned-adm profile latency-performance     # For workstations
tuned-adm profile virtual-guest           # For VMs
```

## Application-Specific Tuning

### Database Tuning

For MySQL/MariaDB:
- Adjust buffer pool size
- Optimize query cache
- Configure innodb_flush_method

### Web Server Tuning

For Apache:
- Adjust MPM settings
- Configure KeepAlive
- Optimize worker processes

For Nginx:
- Configure worker_processes
- Adjust worker_connections
- Optimize keepalive_timeout

## Performance Tuning Best Practices

1. Establish performance baselines
2. Identify bottlenecks through monitoring
3. Make one change at a time
4. Test and measure after each change
5. Document all changes and their effects
6. Consider workload patterns when tuning
7. Use appropriate tuned profiles for your workload