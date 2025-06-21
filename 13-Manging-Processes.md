# Managing Processes in Linux

This document covers process management concepts and commands in Linux systems.

## Understanding Processes

A process is an instance of a running program. Each process in Linux has:
- A unique Process ID (PID)
- A Parent Process ID (PPID)
- User and group ownership
- Resource allocations (CPU, memory, etc.)

## Viewing Processes

### Using `ps` Command

```bash
# List all processes
ps -ef

# List all processes with BSD syntax
ps aux

# List processes in a hierarchical format
ps -ef --forest

# List processes by a specific user
ps -u username
```

### Using `top` and `htop`

```bash
# Interactive process viewer
top

# Enhanced interactive process viewer
htop
```

### Process States

Processes can be in various states:
- R: Running or runnable
- S: Sleeping (interruptible)
- D: Uninterruptible sleep
- Z: Zombie
- T: Stopped or traced

## Process Control

### Starting Processes

```bash
# Start a process in the foreground
command_name

# Start a process in the background
command_name &

# Start a process with modified priority
nice -n 10 command_name
```

### Stopping Processes

```bash
# Terminate a process by PID
kill PID

# Terminate a process by name
pkill process_name

# Force terminate a process
kill -9 PID
```

### Common Signals

- SIGHUP (1): Hangup
- SIGINT (2): Interrupt (Ctrl+C)
- SIGKILL (9): Kill (cannot be caught or ignored)
- SIGTERM (15): Terminate (default signal for kill)
- SIGSTOP (19): Stop (cannot be caught or ignored)

```bash
# Send a specific signal to a process
kill -SIGNAL PID
```

## Process Priorities

### Nice and Renice

Nice values range from -20 (highest priority) to 19 (lowest priority).

```bash
# Start a process with a specific nice value
nice -n value command

# Change the nice value of a running process
renice -n value -p PID
```

## Background and Foreground Jobs

### Job Control

```bash
# List all jobs
jobs

# Bring a background job to the foreground
fg %job_number

# Send a foreground job to the background
Ctrl+Z (to stop) then bg %job_number

# Resume a stopped job in the background
bg %job_number
```

## Process Monitoring and Resource Usage

### Using `pgrep` and `pidof`

```bash
# Find PIDs by process name
pgrep process_name

# Find PID of a specific command
pidof command_name
```

### Using `pstree`

```bash
# Display process tree
pstree

# Display process tree with PIDs
pstree -p
```

### Process Resource Usage

```bash
# Display CPU and memory usage for a specific PID
ps -p PID -o %cpu,%mem,cmd

# Track process resource usage over time
top -p PID
```

## Controlling Process Resource Usage

### Using `cgroups`

Control groups (cgroups) allow you to allocate resources among process groups.

```bash
# Create a cgroup
cgcreate -g subsystem:group_name

# Add a process to a cgroup
cgclassify -g subsystem:group_name PID

# Set resource limits for a cgroup
cgset -r parameter=value group_name
```

### Using `ulimit`

```bash
# View current resource limits
ulimit -a

# Set maximum file size limit
ulimit -f size_in_blocks

# Set maximum number of open files
ulimit -n number
```

## Scheduling Processes

### Using `at` for One-time Tasks

```bash
# Schedule a command to run once at a specific time
at 2:00 PM
command
Ctrl+D

# List scheduled at jobs
atq

# Remove a scheduled at job
atrm job_number
```

### Using `cron` for Recurring Tasks

```bash
# Edit user's crontab
crontab -e

# List user's crontab entries
crontab -l
```

Crontab format: `minute hour day_of_month month day_of_week command`

### Using `systemd` Timers

```bash
# List active timers
systemctl list-timers

# Create a timer unit
# Edit /etc/systemd/system/mytimer.timer and mytimer.service
```

## Troubleshooting Process Issues

### Identifying Resource-Intensive Processes

```bash
# Find top CPU-consuming processes
ps aux --sort=-%cpu | head

# Find top memory-consuming processes
ps aux --sort=-%mem | head
```

### Dealing with Zombie Processes

```bash
# Identify zombie processes
ps aux | grep Z

# Kill parent of zombie process to clean up
kill -HUP parent_PID
```

### Handling Runaway Processes

```bash
# Set resource limits before running a command
ulimit -t 60 && command_name

# Use timeout to limit execution time
timeout 60s command_name
```