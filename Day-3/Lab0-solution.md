# Lab 0: Shared Group Environment Configuration - Solution

This document provides solutions for the shared group environment configuration requirements in Lab 0.

## Solution

### Requirement 1: Set up a shared group environment

Create the directories and set group ownership:

```bash
# Create the directories
mkdir -p /data/account /data/sales

# Set group ownership
chown root:sales /data/sales
chown root:account /data/account
```

### Requirement 2: Configure the permissions

Set permissions for user and group access only (no access for others):

```bash
# Method 1: Using numeric permissions
chmod 770 /data/account
chmod 770 /data/sales

# Method 2: Using symbolic permissions
chmod ugo=rwxrwx--- /data/account
chmod ugo=rwxrwx--- /data/sales
```

### Requirement 3: Configure group ownership inheritance

Set the SGID bit on both directories to ensure group inheritance:

```bash
# Set SGID bit (2) along with rwx for user and group (7), no access for others (0)
chmod 2770 /data/sales
chmod 2770 /data/account
```

### Requirement 4: Configure file deletion restrictions

Set the sticky bit on both directories to restrict file deletion:

```bash
# Add sticky bit to both directories
chmod +t /data/sales
chmod +t /data/account
```

## Verification

To verify the configuration:

```bash
# Check permissions and ownership
ls -la /data/

# Expected output should show:
# drwxrws--T 2 root account ... /data/account
# drwxrws--T 2 root sales ... /data/sales
```

## Explanation

- The `2` in `2770` represents the SGID bit, which ensures that new files created in the directory inherit the group ownership of the directory
- The sticky bit (`+t` or `1` in numeric mode) ensures that only the file owner can delete their own files
- The combined permission `2770` gives full access to user and group, no access to others, and sets the SGID bit
- When both SGID and sticky bit are set, the final numeric permission would be `3770` (3 = 2 + 1)