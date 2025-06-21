# Systemd in Linux

## Introduction to Systemd

Systemd is a system and service manager for Linux operating systems. It has become the standard init system for many major Linux distributions, replacing the traditional SysV init system. Systemd is designed to provide aggressive parallelization capabilities, uses socket and D-Bus activation for starting services, offers on-demand starting of daemons, and implements transactional dependency-based service control logic.

## Key Features of Systemd

- **Service Management**: Start, stop, restart, enable, and disable system services
- **Dependency Handling**: Manage service dependencies automatically
- **Parallel Processing**: Start services in parallel for faster boot times
- **On-Demand Service Activation**: Start services only when needed
- **System State Snapshots**: Create and manage system state snapshots
- **Logging**: Integrated logging with journald
- **Resource Control**: Manage system resources with cgroups
- **Mount and Automount Point Management**: Manage mount points
- **Timer-Based Activation**: Schedule tasks with systemd timers (alternative to cron)

## Systemd Units

Systemd introduces the concept of "units," which are resources that systemd manages. Units are categorized by the type of resource they represent and are defined in unit configuration files.

Common unit types include:

- **Service units** (.service): System services
- **Socket units** (.socket): Inter-process communication sockets
- **Device units** (.device): Hardware devices
- **Mount units** (.mount): File system mount points
- **Automount units** (.automount): Automatic mount points
- **Target units** (.target): Group of units (similar to runlevels)
- **Timer units** (.timer): Scheduled tasks (similar to cron jobs)
- **Slice units** (.slice): Resource management using cgroups
- **Scope units** (.scope): Externally created processes

## Basic Systemd Commands

### Managing Services

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

# Enable a service to start at boot
systemctl enable service_name.service

# Disable a service from starting at boot
systemctl disable service_name.service

# Check if a service is enabled
systemctl is-enabled service_name.service

# Mask a service (prevent it from being started)
systemctl mask service_name.service

# Unmask a service
systemctl unmask service_name.service
```

### System State Management

```bash
# Power off the system
systemctl poweroff

# Reboot the system
systemctl reboot

# Suspend the system
systemctl suspend

# Hibernate the system
systemctl hibernate

# Put system into hybrid sleep
systemctl hybrid-sleep
```

### Viewing System Status

```bash
# View system status
systemctl status

# List all running units
systemctl list-units

# List all failed units
systemctl --failed

# List all installed unit files
systemctl list-unit-files

# List dependencies of a unit
systemctl list-dependencies service_name.service
```

## Working with Unit Files

Unit files are typically stored in the following locations:
- `/usr/lib/systemd/system/`: Units provided by installed packages
- `/etc/systemd/system/`: Units installed by the system administrator
- `/run/systemd/system/`: Runtime units

### Viewing Unit Files

```bash
# View a unit file
systemctl cat service_name.service

# Show properties of a unit
systemctl show service_name.service

# Edit a unit file
systemctl edit service_name.service

# Edit the full unit file
systemctl edit --full service_name.service
```

### Creating a Custom Service Unit

Create a file in `/etc/systemd/system/myservice.service`:

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/myprogram --option1
Restart=on-failure
User=myuser
Group=mygroup

[Install]
WantedBy=multi-user.target
```

After creating or modifying unit files, reload the systemd manager:

```bash
systemctl daemon-reload
```

## Understanding Unit File Sections

### [Unit] Section

Contains metadata and dependencies:

```ini
[Unit]
Description=My Service Description
Documentation=man:myservice(8) http://example.com/docs
Requires=network.target
After=network.target mysql.service
Conflicts=conflicting-service.service
```

### [Service] Section

Configures service-specific behavior:

```ini
[Service]
Type=simple
ExecStart=/usr/bin/myprogram
ExecStartPre=/usr/bin/pre-start-script
ExecStartPost=/usr/bin/post-start-script
ExecStop=/usr/bin/stop-script
ExecReload=/usr/bin/reload-script
Restart=on-failure
RestartSec=5s
TimeoutStartSec=30s
TimeoutStopSec=30s
User=serviceuser
Group=servicegroup
WorkingDirectory=/var/lib/myservice
Environment="VAR1=value1" "VAR2=value2"
EnvironmentFile=/etc/myservice/env
```

### [Install] Section

Defines how the unit should be installed:

```ini
[Install]
WantedBy=multi-user.target
RequiredBy=another-service.service
Alias=myservice.service myalias.service
Also=additional-service.service
```

## Systemd Targets

Targets are groups of units that provide a way to bring the system to a specific state. They are similar to runlevels in SysV init.

Common targets include:

- **poweroff.target**: Shutdown system
- **rescue.target**: Single-user mode
- **multi-user.target**: Multi-user, non-graphical system
- **graphical.target**: Multi-user, graphical system
- **reboot.target**: Reboot system

### Managing Targets

```bash
# View current target
systemctl get-default

# Set default target
systemctl set-default multi-user.target

# Switch to a target
systemctl isolate multi-user.target

# List available targets
systemctl list-units --type=target
```

## Systemd Journal

Systemd includes a centralized logging service called journald, which collects and stores logging data.

```bash
# View all logs
journalctl

# View logs for a specific service
journalctl -u service_name.service

# View kernel messages
journalctl -k

# View logs since last boot
journalctl -b

# View logs in real-time (follow)
journalctl -f

# View logs within a time range
journalctl --since="2023-01-01" --until="2023-01-02"

# View logs with specific priority
journalctl -p err
```

## Systemd Timers

Systemd timers provide an alternative to cron for scheduling tasks.

### Creating a Timer

Create two files:

1. The service file (`/etc/systemd/system/mytask.service`):
```ini
[Unit]
Description=My Scheduled Task

[Service]
Type=oneshot
ExecStart=/path/to/my/script.sh
User=myuser
```

2. The timer file (`/etc/systemd/system/mytask.timer`):
```ini
[Unit]
Description=Run My Task Every Day

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Managing Timers

```bash
# Enable and start the timer
systemctl enable mytask.timer
systemctl start mytask.timer

# List all timers
systemctl list-timers

# Check timer status
systemctl status mytask.timer
```

## Resource Management with Systemd

Systemd uses cgroups to manage resources for services:

```ini
[Service]
# Limit CPU usage
CPUQuota=50%
CPUWeight=800

# Limit memory usage
MemoryLimit=1G

# Limit I/O
IOWeight=500
IODeviceWeight=/dev/sda 800

# Limit tasks
TasksMax=100
```

## Troubleshooting Systemd

### Common Issues and Solutions

1. **Service fails to start**:
   - Check service status: `systemctl status service_name.service`
   - Check logs: `journalctl -u service_name.service`
   - Verify unit file syntax: `systemd-analyze verify service_name.service`

2. **Dependency problems**:
   - Check dependencies: `systemctl list-dependencies service_name.service`
   - Ensure required services are running

3. **Permission issues**:
   - Check file permissions for executables
   - Verify user/group settings in unit file

4. **Timeout issues**:
   - Adjust timeout settings in unit file
   - Check for resource constraints

### Systemd Analysis Tools

```bash
# Analyze boot time
systemd-analyze

# Analyze boot time by units
systemd-analyze blame

# Create a critical chain analysis
systemd-analyze critical-chain

# Verify unit file syntax
systemd-analyze verify unit_file.service
```

## Best Practices for Systemd

1. **Use descriptive unit names and descriptions**
2. **Set appropriate dependencies**
3. **Configure proper restart behavior**
4. **Use resource limits to prevent service abuse**
5. **Implement proper logging**
6. **Test unit files before deployment**
7. **Document custom unit files**
8. **Use override files instead of editing vendor unit files**
9. **Regularly review service status and logs**
10. **Keep systemd updated to benefit from bug fixes and new features**