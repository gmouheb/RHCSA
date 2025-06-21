# Lab 1: System Performance Monitoring - Solution

This document provides solutions for the system performance monitoring tasks in Lab 1.

## Solution

To assess your system's performance, you can use several built-in Linux utilities. Here are the key tools for monitoring different aspects of system performance:

### 1. CPU Usage and Load Average

#### Using top

The `top` command provides a dynamic real-time view of system performance:

```bash
top
```

Key metrics to check in the output:
- **Load average**: The three numbers show the average system load over 1, 5, and 15 minutes
  - Values below the number of CPU cores indicate good performance
  - Values consistently above the number of cores indicate CPU saturation
- **CPU usage percentages**: Look for high values in the %us (user), %sy (system), or %wa (I/O wait) columns

#### Using uptime

For a quick check of load average:

```bash
uptime
```

#### Using mpstat

To see CPU statistics by processor:

```bash
mpstat -P ALL
```

### 2. Memory Utilization

#### Using free

Check memory usage with:

```bash
free -h
```

Key metrics:
- Available memory
- Swap usage (high swap usage with available memory indicates potential issues)

#### Using vmstat

For memory and virtual memory statistics:

```bash
vmstat 1 5
```

### 3. Disk I/O Performance

#### Using iostat

Monitor disk I/O statistics:

```bash
iostat -xz 1 5
```

Key metrics:
- %util: Percentage of CPU time during which I/O requests were issued
- await: Average time for I/O requests to be serviced (in milliseconds)

#### Using df

Check disk space usage:

```bash
df -h
```

### 4. Network Activity

#### Using netstat or ss

View network connections:

```bash
ss -tuln
# or
netstat -tuln
```

#### Using iftop

Monitor bandwidth usage on an interface (may need to be installed):

```bash
iftop -i eth0
```

#### Using nload

Another bandwidth monitoring tool (may need to be installed):

```bash
nload
```

## Interpreting Results

A system in good shape typically shows:

1. **CPU**: Load average below the number of CPU cores, low %wa (wait) time
2. **Memory**: Sufficient available memory, minimal swap usage
3. **Disk**: Low %util and await times, adequate free space
4. **Network**: No excessive packet drops or errors

## Example Performance Report

```
System Performance Assessment:
- CPU: Load average 0.45, 0.32, 0.28 on a 4-core system (Good)
- Memory: 6.2G used of 16G total, 0 swap used (Good)
- Disk: sda running at 12% utilization, 2.3ms await time (Good)
- Network: eth0 showing normal traffic patterns, no errors (Good)

Conclusion: System is performing well with no apparent bottlenecks.
```

## Additional Tools

For more comprehensive monitoring, consider:
- `sar` - Collect and report system activity information
- `htop` - An enhanced version of top with a more user-friendly interface
- `glances` - A cross-platform monitoring tool
- `iotop` - Monitor disk I/O usage by processes
- `nmon` - Performance monitoring tool for Linux