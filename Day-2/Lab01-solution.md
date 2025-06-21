# Lab 1: Configuring Sudo Permissions - Solution

This document provides the solution for the sudo configuration task in Lab 1.

## Solution

To configure sudo permissions for user bill that allow managing user properties and passwords but prevent changing the root password:

1. Edit the sudoers file using the visudo command:

```bash
sudo visudo
```

2. Add the following line to the sudoers file:

```
bill ALL=(ALL) /usr/sbin/usermod, /usr/sbin/useradd, /usr/sbin/userdel, /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd root
```

## Explanation

This sudo configuration:

- Allows user `bill` to run specific user management commands (`usermod`, `useradd`, `userdel`) with root privileges
- Allows `bill` to change passwords for any user with a username containing letters (`passwd [A-Za-z]*`)
- Explicitly denies `bill` from changing the root password (`!/usr/bin/passwd root`)

The `!` symbol is used to create an exclusion rule, which takes precedence over the more general permission.

## Verification

To verify the configuration works correctly, you can:

1. Switch to the bill user:
```bash
su - bill
```

2. Try to modify a regular user:
```bash
sudo passwd someuser
```
This should work.

3. Try to change the root password:
```bash
sudo passwd root
```
This should be denied with a permission error.