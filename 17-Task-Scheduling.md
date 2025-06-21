# Task Scheduling in Linux

This document covers various methods for scheduling tasks in Linux systems.

## Introduction to Task Scheduling

Task scheduling allows administrators to automate routine tasks, maintenance jobs, and other processes to run at specified times without manual intervention.

Linux provides several tools for task scheduling:
- cron
- at
- batch
- systemd timers

## Scheduling with Cron

Cron is the traditional and most widely used task scheduling utility in Linux.

### Cron Directories

Cron configuration files are stored in several locations:
- `/etc/crontab` - System-wide crontab file
- `/etc/cron.d/` - Directory for additional crontab files
- `/etc/cron.hourly/` - Scripts run hourly
- `/etc/cron.daily/` - Scripts run daily
- `/etc/cron.weekly/` - Scripts run weekly
- `/etc/cron.monthly/` - Scripts run monthly
- `/var/spool/cron/` - User-specific crontab files

### Crontab Format

The format of a crontab entry is:
```
# Minute Hour Day Month DayOfWeek Command
```

For example:
```
# Run at 2:30 AM every day
30 2 * * * /path/to/command

# Run every 15 minutes
*/15 * * * * /path/to/command

# Run at 6:00 PM on weekdays
0 18 * * 1-5 /path/to/command

# Run at midnight on the first day of each month
0 0 1 * * /path/to/command
```

### Special Cron Time Strings

Cron also supports special time strings:
- `@reboot` - Run once at startup
- `@yearly` or `@annually` - Run once a year (0 0 1 1 *)
- `@monthly` - Run once a month (0 0 1 * *)
- `@weekly` - Run once a week (0 0 * * 0)
- `@daily` or `@midnight` - Run once a day (0 0 * * *)
- `@hourly` - Run once an hour (0 * * * *)

### Managing Crontabs

```bash
# Edit your crontab
crontab -e

# List your crontab entries
crontab -l

# Remove your crontab
crontab -r

# Edit another user's crontab (as root)
crontab -u username -e
```

### Example Crontab Entries

```
# Run a backup script at 3:00 AM every day
0 3 * * * /usr/local/bin/backup.sh

# Run a cleanup script at 2:00 AM every Monday
0 2 * * 1 /usr/local/bin/cleanup.sh

# Run a script every 5 minutes
*/5 * * * * /usr/local/bin/check-service.sh

# Run a script at the beginning of every month
0 0 1 * * /usr/local/bin/monthly-report.sh

# Run a script at system startup
@reboot /usr/local/bin/startup-script.sh
```

## Scheduling with At

The `at` command is used for scheduling one-time tasks to run at a specific time.

```bash
# Install at if not already installed
dnf install at

# Enable and start the atd service
systemctl enable --now atd
```

### Using At Command

```bash
# Schedule a command to run at a specific time
at 2:30 PM
command1
command2
Ctrl+D

# Schedule a command to run in the future
at now + 1 hour
command
Ctrl+D

# Schedule a command to run on a specific date
at 10:00 AM July 25
command
Ctrl+D
```

### Managing At Jobs

```bash
# List pending at jobs
atq

# View details of a specific job
at -c job_number

# Remove a scheduled job
atrm job_number
```

## Scheduling with Batch

The `batch` command is similar to `at` but runs jobs when system load permits.

```bash
# Schedule a job to run when system load is low
batch
command
Ctrl+D
```

## Scheduling with Systemd Timers

Systemd timers provide an alternative to cron with better logging and dependency handling.

### Creating a Systemd Timer

1. Create a service unit file (e.g., `/etc/systemd/system/mytask.service`):
```ini
[Unit]
Description=My Scheduled Task

[Service]
Type=oneshot
ExecStart=/path/to/script.sh
User=username
```

2. Create a timer unit file (e.g., `/etc/systemd/system/mytask.timer`):
```ini
[Unit]
Description=Run My Scheduled Task

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Timer Specifications

Systemd timers support various time specifications:

- `OnCalendar=`: Calendar event expressions
  - `*-*-* 02:00:00` - Every day at 2:00 AM
  - `Mon *-*-* 03:00:00` - Every Monday at 3:00 AM
  - `*-*-01 04:00:00` - First day of every month at 4:00 AM

- `OnBootSec=`: Time relative to boot
  - `OnBootSec=10min` - 10 minutes after boot

- `OnUnitActiveSec=`: Time relative to last activation
  - `OnUnitActiveSec=1h` - Every hour after last run

### Managing Systemd Timers

```bash
# Enable and start a timer
systemctl enable mytask.timer
systemctl start mytask.timer

# Check timer status
systemctl status mytask.timer

# List all active timers
systemctl list-timers

# List all timers (including inactive)
systemctl list-timers --all
```

## Anacron - For Systems Not Running 24/7

Anacron ensures that periodic jobs run even if the system is not running continuously.

```bash
# Install anacron
dnf install anacron
```

### Anacron Configuration

The main configuration file is `/etc/anacrontab`:

```
# period  delay  job-identifier  command
1         5      cron.daily      run-parts /etc/cron.daily
7         10     cron.weekly     run-parts /etc/cron.weekly
30        15     cron.monthly    run-parts /etc/cron.monthly
```

### Custom Anacron Jobs

```
# Run a script daily with a 10-minute delay
1  10  my-daily-job  /path/to/script.sh
```

## Controlling Access to Scheduling Tools

### Cron Access Control

The files `/etc/cron.allow` and `/etc/cron.deny` control who can use cron:

- If `cron.allow` exists, only users listed in it can use cron
- If `cron.allow` doesn't exist but `cron.deny` does, all users except those listed in `cron.deny` can use cron
- If neither file exists, only root can use cron (on some systems, all users can use it)

### At Access Control

Similarly, `/etc/at.allow` and `/etc/at.deny` control access to the `at` command.

## Logging and Troubleshooting

### Cron Logs

Cron logs are typically found in:
- `/var/log/cron`
- System journal (`journalctl -u crond.service`)

### Systemd Timer Logs

```bash
# View logs for a specific timer
journalctl -u mytask.timer

# View logs for the associated service
journalctl -u mytask.service
```

### Common Issues and Solutions

1. **Script not running:**
   - Check script permissions (must be executable)
   - Verify the path to the script is correct
   - Check for syntax errors in the script
   - Ensure the script has a proper shebang line (e.g., `#!/bin/bash`)

2. **Environment variables:**
   - Cron runs with a limited environment
   - Define all required environment variables in the script or crontab

3. **Email notifications:**
   - Configure a mail transfer agent (MTA) to receive cron job output
   - Redirect output to a file: `0 3 * * * /path/to/script.sh > /path/to/log 2>&1`