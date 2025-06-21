# Lab 3: User Management and Process Priority - Solution

This document provides solutions for the user management and process priority tasks in Lab 3.

## Solution

### Task 1: Create User and Open Shell

Create the user linda and switch to that user:

```bash
# Create the user
sudo useradd -m linda

# Set a password for the user (required for login)
sudo passwd linda

# Switch to the linda user
su - linda
```

### Task 2: Run Background Processes with Different Priorities

As user linda, run two sleep processes with different priorities:

```bash
# Run a process with the lowest possible priority (nice value 19)
nice -n 19 sleep 600 &

# For the highest priority, regular users can only use nice values 0-19
# To get the highest priority a regular user can set (nice value 0):
sleep 600 &
```

Note: Regular users can only decrease priority (increase nice value from 0 to 19). Only root can set negative nice values for higher priorities. If you want to truly set the highest possible priority (-20), you would need to do it as root:

```bash
# Only root can do this:
sudo nice -n -20 sleep 600
```

You can verify the nice values with:

```bash
ps -o pid,comm,nice -p $(pgrep sleep)
```

### Task 3: Terminate All User Sessions

To terminate all processes owned by user linda (which effectively terminates all sessions):

```bash
# As root or with sudo
sudo pkill -u linda
```

This command sends the SIGTERM signal to all processes owned by linda. If some processes don't terminate, you can use the SIGKILL signal:

```bash
sudo pkill -9 -u linda
```

## Additional Information

### Understanding Nice Values

In Linux, process priority is controlled by the "nice" value:
- Range is from -20 (highest priority) to 19 (lowest priority)
- Default nice value is 0
- Regular users can only increase nice values (lower priority)
- Only root can set negative nice values (higher priority)

### Checking User Sessions

To check active user sessions before terminating them:

```bash
w
# or
who
```

### Alternative Methods to Terminate Sessions

Besides `pkill -u`, you can also use:

```bash
# Kill all processes owned by linda
sudo killall -u linda

# More forceful termination
sudo skill -KILL -u linda
```