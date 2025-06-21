# Lab 0: Basic Shell Operations - Solutions

This document provides solutions for the tasks in Lab 0.

## Solutions

### Task 1: Setting Environment Variable
To set the LOGGEDUSER variable in every subshell:

```bash
# Edit your bash configuration file
nano ~/.bashrc

# Add one of these lines:
export LOGGEDUSER='user'
# or
LOGGEDUSER='user'

# Apply the changes to current session
source ~/.bashrc

# Verify it's working
echo $LOGGEDUSER
```

### Task 2: Finding Password Change Command
To find the command for changing passwords:

```bash
# Method 1 (best approach)
which passwd

# Alternative methods
sudo updatedb
locate passwd
```

You can use the command as:
```bash
passwd           # Change your own password
sudo passwd      # Change another user's password (requires root privileges)
```

Yes, you need root permissions to change another user's password, but not to change your own.

### Task 3: Redirecting Output and Errors
To redirect both standard output and errors to a file:

```bash
ls -al wergihl* > /tmp/lsoutput 2>&1
```

This command redirects both standard output (file descriptor 1) and standard error (file descriptor 2) to the file /tmp/lsoutput.