# Lab 2: Process Management and System Tuning - Solution

This document provides solutions for the process management and system tuning tasks in Lab 2.

## Solution

### Task 1: Launch Background Jobs

To launch three instances of the `dd` command as background jobs:

```bash
dd if=/dev/zero of=/dev/null &
dd if=/dev/zero of=/dev/null &
dd if=/dev/zero of=/dev/null &
```

Each command will return a job number and process ID. For example:
```
[1] 12345
[2] 12346
[3] 12347
```

You can verify the running processes with:
```bash
ps aux | grep dd
```

### Task 2: Adjust Process Priority

First, identify the process ID (PID) of one of the dd commands:

```bash
ps aux | grep "dd if=/dev/zero"
```

This will show output similar to:
```
user     12345  0.5  0.0   4508   800 pts/0    R    10:15   0:02 dd if=/dev/zero of=/dev/null
user     12346  0.5  0.0   4508   800 pts/0    R    10:15   0:02 dd if=/dev/zero of=/dev/null
user     12347  0.5  0.0   4508   800 pts/0    R    10:15   0:02 dd if=/dev/zero of=/dev/null
```

Now, change the priority using the `renice` command with a value of -5 (note: negative values increase priority and require root privileges):

```bash
sudo renice -5 -p 12345
```

Check the new priority:
```bash
ps -o pid,ni,cmd -p 12345
```

Change the priority again to -15:
```bash
sudo renice -15 -p 12345
```

Check the priority again:
```bash
ps -o pid,ni,cmd -p 12345
```

Observe the difference in CPU usage with `top` or `htop`. The process with the higher priority (more negative nice value) will get more CPU time when the system is under load.

### Task 3: Kill the Processes

To kill all dd processes:

```bash
pkill dd
```

Verify that they are terminated:
```bash
ps aux | grep dd
```

### Task 4: Configure Tuned

Install tuned if not already installed:

```bash
sudo dnf -y install tuned
```

Enable and start the tuned service:

```bash
sudo systemctl enable --now tuned
```

List available profiles:

```bash
tuned-adm list
```

Set the throughput-performance profile:

```bash
sudo tuned-adm profile throughput-performance
```

Verify the active profile:

```bash
tuned-adm active
```

The output should confirm that the throughput-performance profile is active.

## Understanding Process Priority

In Linux, process priority is controlled by the "nice" value:
- The default nice value is 0
- Range is from -20 (highest priority) to 19 (lowest priority)
- Only root can set negative nice values
- Higher priority processes get more CPU time when the system is busy

## Understanding Tuned Profiles

The tuned service provides various predefined profiles optimized for different workloads:
- **throughput-performance**: Optimized for high throughput at the expense of latency
- **latency-performance**: Optimized for low latency at the expense of throughput
- **balanced**: Default profile that balances performance and energy efficiency
- **powersave**: Optimized for energy efficiency at the expense of performance

The throughput-performance profile is particularly useful for server workloads where maximum processing capacity is more important than response time.