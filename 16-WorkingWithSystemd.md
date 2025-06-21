# Working with Systemd in Linux

This document covers systemd, the system and service manager for Linux operating systems.

## Introduction to Systemd

Systemd is the init system and service manager for most modern Linux distributions. It replaces the traditional SysV init system and provides:

- Parallel startup of system services
- On-demand service activation
- Dependency management between services
- Service state tracking
- Logging with journald
- System resource management

## Systemd Units

Systemd organizes its resources as "units," which can be of various types:

- **service**: System services
- **socket**: IPC sockets for service activation
- **target**: Groups of units
- **device**: Device units
- **mount**: Filesystem mount points
- **automount**: Automount points
- **timer**: Timer-based activation
- **swap**: Swap partitions or files
- **path**: Filesystem path monitoring
- **slice**: Resource management slices
- **scope**: Externally created processes

## Managing Services with Systemd

### Basic Service Management

```bash
# Start a service
systemctl start service_name.service

# Stop a service
systemctl stop service_name.service

# Restart a service
systemctl restart service_name.service

# Reload a service's configuration
systemctl reload service_name.service

# Check service status
systemctl status service_name.service
```

### Enabling and Disabling Services

```bash
# Enable a service to start at boot
systemctl enable service_name.service

# Disable a service from starting at boot
systemctl disable service_name.service

# Check if a service is enabled
systemctl is-enabled service_name.service

# Enable and immediately start a service
systemctl enable --now service_name.service

# Disable and immediately stop a service
systemctl disable --now service_name.service
```

### Masking and Unmasking Services

```bash
# Mask a service (prevent it from being started)
systemctl mask service_name.service

# Unmask a service
systemctl unmask service_name.service
```

## Viewing System State

```bash
# View system boot status
systemctl status

# List all active units
systemctl list-units

# List all units of a specific type
systemctl list-units --type=service

# List all failed units
systemctl list-units --state=failed

# List all installed unit files
systemctl list-unit-files

# List dependencies of a unit
systemctl list-dependencies service_name.service
```

## Working with Targets

Targets are groups of units that represent system states (similar to runlevels in SysV init).

```bash
# View current target
systemctl get-default

# Set default target
systemctl set-default target_name.target

# Switch to a different target
systemctl isolate target_name.target

# List available targets
systemctl list-units --type=target
```

Common targets:
- **multi-user.target**: Multi-user system without GUI (runlevel 3)
- **graphical.target**: Multi-user system with GUI (runlevel 5)
- **rescue.target**: Single-user mode (runlevel 1)
- **emergency.target**: Emergency shell

## Creating Custom Service Units

Service unit files are typically stored in:
- `/usr/lib/systemd/system/` (system packages)
- `/etc/systemd/system/` (local administrator)

### Example Service Unit File

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/my-service
ExecStop=/usr/local/bin/my-service --stop
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Service Types

- **simple**: Main process is the process started as ExecStart
- **forking**: Process forks and parent exits
- **oneshot**: Process exits before starting follow-up units
- **dbus**: Process acquires a D-Bus name
- **notify**: Process sends notification when ready
- **idle**: Similar to simple, but delayed until system is idle

### After Creating or Modifying Unit Files

```bash
# Reload systemd manager configuration
systemctl daemon-reload

# Start and test the new service
systemctl start my-service.service
systemctl status my-service.service
```

## Working with Systemd Timers

Systemd timers can be used as alternatives to cron jobs.

```bash
# List all timers
systemctl list-timers

# Create a timer unit file (example: /etc/systemd/system/my-task.timer)
# [Unit]
# Description=Run my task daily
#
# [Timer]
# OnCalendar=*-*-* 02:00:00
# Persistent=true
#
# [Install]
# WantedBy=timers.target

# Create corresponding service unit (example: /etc/systemd/system/my-task.service)
# [Unit]
# Description=My daily task
#
# [Service]
# Type=oneshot
# ExecStart=/usr/local/bin/my-script.sh
```

## Systemd Journal

Systemd includes a centralized logging service called journald.

```bash
# View all journal entries
journalctl

# View journal entries for a specific service
journalctl -u service_name.service

# View journal entries since last boot
journalctl -b

# View journal entries for a specific time period
journalctl --since="2023-01-01" --until="2023-01-02"

# Follow new journal entries (like tail -f)
journalctl -f

# View journal entries by priority
journalctl -p err
```

## Resource Management with Systemd

Systemd can manage system resources using cgroups.

```bash
# Set resource limits for a service
# Edit or create a drop-in configuration:
# /etc/systemd/system/service_name.service.d/limits.conf
#
# [Service]
# CPUQuota=20%
# MemoryLimit=1G
# IOWeight=500

# Apply changes
systemctl daemon-reload
systemctl restart service_name.service
```

## Analyzing System Boot Performance

```bash
# View system boot time
systemd-analyze

# View time spent by each service during boot
systemd-analyze blame

# Create a visualization of the boot process
systemd-analyze plot > boot.svg
```

## Troubleshooting Systemd Services

```bash
# Check if a service is running
systemctl is-active service_name.service

# View detailed service status
systemctl status service_name.service

# View service logs
journalctl -u service_name.service

# View service logs for the current boot
journalctl -u service_name.service -b

# Check service configuration
systemctl show service_name.service

# List service dependencies
systemctl list-dependencies service_name.service
```