# User Configuration and Permissions in Linux

This guide covers the essential concepts of user configuration files, permission settings, and UMASK in Linux systems.

## System Configuration Files

### Global User Configuration

The `/etc/login.defs` file contains system-wide settings that affect all users, including:

- Password aging controls
- User and group ID ranges
- Encryption methods for passwords
- Mail directory settings
- UMASK defaults
- Login and session settings

Example settings in `/etc/login.defs`:
```
PASS_MAX_DAYS   99999
PASS_MIN_DAYS   0
PASS_MIN_LEN    5
PASS_WARN_AGE   7
UID_MIN         1000
UID_MAX         60000
GID_MIN         1000
GID_MAX         60000
UMASK           022
CREATE_HOME     yes
```

### User-Specific Configuration Files

Individual user settings are stored in the user's home directory:

- `~/.bashrc`: Contains user-specific shell settings loaded for interactive non-login shells
- `~/.bash_profile`: Contains user-specific settings loaded for login shells
- `~/.profile`: Used if `.bash_profile` doesn't exist
- `~/.bash_logout`: Commands executed when the user logs out

## Understanding UMASK

UMASK (User Mask) determines the default permissions for newly created files and directories.

### Default UMASK Setting

The default UMASK is typically `022`, which means:
- Files are created with permissions `644` (rw-r--r--)
- Directories are created with permissions `755` (rwxr-xr-x)

### How UMASK Works

UMASK works by *subtracting* from the maximum default permissions:
- Maximum default for files: `666` (rw-rw-rw-)
- Maximum default for directories: `777` (rwxrwxrwx)

### UMASK Calculation

To calculate the resulting permissions:
1. Start with the default maximum permissions (666 for files, 777 for directories)
2. Subtract the UMASK value

#### Example Calculation with UMASK 022:

For files:
```
666 - 022 = 644 (rw-r--r--)
```

For directories:
```
777 - 022 = 755 (rwxr-xr-x)
```

### Setting UMASK

To temporarily change the UMASK for your current session:
```bash
umask 027  # More restrictive
```

To permanently change the UMASK for a user, add the command to their `.bashrc` or `.bash_profile`:
```bash
echo "umask 027" >> ~/.bashrc
```

To change the system-wide default, edit `/etc/login.defs` or `/etc/profile`.

## File Permissions

### Permission Symbols

File permissions in Linux are represented by symbols:
- `r`: Read permission
- `w`: Write permission
- `x`: Execute permission

### User Classes (UGO)

Permissions are assigned to three classes of users:
- `u`: User (owner of the file)
- `g`: Group (users in the group that owns the file)
- `o`: Other (users who are neither the owner nor part of the group)

### Symbolic Representation

File permissions are displayed in the format:
```
-rwxrw-r--
```

Breaking this down:
- First character: File type (`-` for regular file, `d` for directory)
- Next three characters: Owner permissions (`rwx`)
- Next three characters: Group permissions (`rw-`)
- Last three characters: Others permissions (`r--`)

### Numeric Permission Values

Permissions can also be represented numerically:
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

Adding these values gives the permission for each user class:
- `rwx` = 7 (4+2+1)
- `rw-` = 6 (4+2)
- `r-x` = 5 (4+1)
- `r--` = 4 (4)
- `-wx` = 3 (2+1)
- `-w-` = 2 (2)
- `--x` = 1 (1)
- `---` = 0 (0)

### Common Permission Settings

- `777` (rwxrwxrwx): Full permissions for everyone (rarely appropriate)
- `755` (rwxr-xr-x): Owner can do anything; others can read and execute
- `750` (rwxr-x---): Owner can do anything; group can read and execute; others have no access
- `700` (rwx------): Only owner has full access; no one else has any access
- `644` (rw-r--r--): Owner can read and write; others can only read
- `640` (rw-r-----): Owner can read and write; group can read; others have no access
- `600` (rw-------): Only owner can read and write; no one else has any access

## Changing Permissions

### Using chmod with Symbolic Notation

```bash
chmod u+x file.sh       # Add execute permission for owner
chmod g-w file.txt      # Remove write permission for group
chmod o=r file.txt      # Set other permissions to read-only
chmod a+r file.txt      # Add read permission for all users
```

### Using chmod with Numeric Notation

```bash
chmod 755 file.sh       # rwxr-xr-x
chmod 644 file.txt      # rw-r--r--
chmod 600 private.key   # rw-------
```

## Special Permission Bits

In addition to the basic permissions, Linux supports special permission bits:

- `setuid` (4000): When set on an executable file, it runs with the permissions of the file owner
- `setgid` (2000): When set on an executable file, it runs with the permissions of the file group
- `sticky bit` (1000): When set on a directory, only the file owner can delete or rename files

Example:
```bash
chmod 4755 file         # Set setuid bit (rwsr-xr-x)
chmod 2755 directory    # Set setgid bit (rwxr-sr-x)
chmod 1777 /tmp         # Set sticky bit (rwxrwxrwt)
```

## Best Practices

1. Use the principle of least privilege when setting permissions
2. Regularly audit file permissions, especially for sensitive files
3. Be cautious with setuid and setgid permissions
4. Consider using more restrictive UMASK settings (027 or 077) for enhanced security
5. Use Access Control Lists (ACLs) for more fine-grained permission control
6. Group users with similar access needs into appropriate groups