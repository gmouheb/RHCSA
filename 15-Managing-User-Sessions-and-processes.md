# Managing User Sessions and Processes in Linux

This document covers the management of user sessions and their associated processes in Linux systems.

## Understanding User Sessions

A user session begins when a user logs into a system and ends when they log out. During a session, the user can run multiple processes.

### Types of User Sessions

- **Local sessions**: Direct login through a physical terminal
- **Remote sessions**: SSH, telnet, or other remote access protocols
- **GUI sessions**: X11, Wayland, or other graphical environments
- **Non-interactive sessions**: Cron jobs, system services

## Viewing User Sessions

### Using `who` and `w` Commands

```bash
# Display who is logged in
who

# Display who is logged in and what they're doing
w

# Display your own username
whoami

# Display last logged in users
last

# Display failed login attempts
lastb
```

### Using `loginctl` (systemd)

```bash
# List all sessions
loginctl list-sessions

# Show details of a specific session
loginctl show-session SESSION_ID

# List all users with active sessions
loginctl list-users

# Show details of a specific user
loginctl show-user USERNAME
```

## Managing User Sessions

### Terminating User Sessions

```bash
# Terminate a specific session
loginctl terminate-session SESSION_ID

# Kill all sessions of a specific user
loginctl kill-user USERNAME

# Force logout a user (old method)
pkill -KILL -u USERNAME
```

### Session Limits and Controls

```bash
# Set resource limits for user sessions in /etc/security/limits.conf
# Example:
# username hard nproc 100  # Limit max processes
# username hard maxlogins 2  # Limit concurrent logins
```

## Process Management by User

### Viewing User Processes

```bash
# List processes for a specific user
ps -u USERNAME

# List processes for a specific user with full details
ps -f -u USERNAME

# List processes for multiple users
ps -f -u USERNAME1,USERNAME2

# List processes sorted by CPU usage
ps -u USERNAME --sort=-%cpu

# List processes sorted by memory usage
ps -u USERNAME --sort=-%mem
```

### Controlling User Processes

```bash
# Kill all processes owned by a user
pkill -u USERNAME

# Send a specific signal to user processes
pkill -SIGNAL -u USERNAME

# Kill a specific process by PID
kill PID

# Force kill a process
kill -9 PID
```

## Resource Limits for Users

### Using `ulimit`

```bash
# View current resource limits
ulimit -a

# Set maximum file size limit
ulimit -f SIZE_IN_BLOCKS

# Set maximum number of processes
ulimit -u NUMBER_OF_PROCESSES

# Set maximum number of open files
ulimit -n NUMBER_OF_FILES
```

### Persistent Limits in `/etc/security/limits.conf`

```
# Format: <domain> <type> <item> <value>
# Example:
# username soft nproc 1024
# username hard nproc 2048
# @group soft nofile 4096
# @group hard nofile 8192
```

## User Process Accounting

### Enabling Process Accounting

```bash
# Install the psacct package
dnf install psacct

# Enable and start the service
systemctl enable psacct
systemctl start psacct
```

### Using Process Accounting Commands

```bash
# Display statistics about users' connect time
ac

# Display statistics about commands executed by users
sa

# Display information about last commands executed by users
lastcomm

# Display information about a specific command
lastcomm command_name

# Display information about commands executed by a specific user
lastcomm username
```

## Session and Process Monitoring

### Real-time Monitoring

```bash
# Monitor user activity in real-time
watch -n 1 "w"

# Monitor processes in real-time
top -u USERNAME

# Enhanced process monitoring
htop -u USERNAME
```

### Logging and Auditing

```bash
# Configure audit rules for user activity
auditctl -w /etc/passwd -p wa -k user-modification

# Search audit logs for user activity
ausearch -k user-modification

# Generate reports from audit logs
aureport --auth
```

## Managing User Sessions with PAM

PAM (Pluggable Authentication Modules) can be used to control user sessions.

```bash
# Common PAM session modules:
# pam_limits.so - Apply resource limits
# pam_unix.so - Standard Unix session management
# pam_systemd.so - Register user sessions with systemd
```

Example PAM configuration in `/etc/pam.d/system-auth`:
```
session     required      pam_limits.so
session     required      pam_unix.so
session     optional      pam_systemd.so
```

## Best Practices for User Session Management

1. Implement session timeouts for inactive users
2. Set appropriate resource limits for different user classes
3. Regularly monitor and audit user sessions
4. Use PAM to enforce session policies
5. Configure automatic session termination for suspicious activities
6. Implement proper logging for session activities
7. Educate users about session management and security