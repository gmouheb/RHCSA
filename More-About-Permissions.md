# More About Permissions in Linux

This document covers advanced permission concepts and techniques in Linux systems.

## Beyond Basic Permissions

While the standard Linux permissions (read, write, execute for user, group, and others) provide basic access control, Linux offers several advanced permission mechanisms:

- Special permissions (setuid, setgid, sticky bit)
- Access Control Lists (ACLs)
- Extended attributes
- Capabilities
- Mandatory Access Control (MAC) systems like SELinux and AppArmor

## Special Permissions

### SetUID (Set User ID)

The SetUID permission allows a user to execute a file with the permissions of the file's owner.

```bash
# Set the SetUID bit
chmod u+s /path/to/file
# or
chmod 4755 /path/to/file

# View files with SetUID bit
find / -perm -4000 -type f -exec ls -l {} \; 2>/dev/null
```

Common SetUID programs:
- `/usr/bin/passwd` - Allows users to change their passwords
- `/usr/bin/sudo` - Allows authorized users to execute commands as another user
- `/usr/bin/ping` - Allows sending ICMP packets which requires raw socket access

### SetGID (Set Group ID)

The SetGID permission has two effects:
1. On executable files: Allows execution with the permissions of the file's group
2. On directories: New files created in the directory inherit the directory's group

```bash
# Set the SetGID bit on a file
chmod g+s /path/to/file
# or
chmod 2755 /path/to/file

# Set the SetGID bit on a directory
chmod g+s /path/to/directory
# or
chmod 2775 /path/to/directory

# View files with SetGID bit
find / -perm -2000 -type f -exec ls -l {} \; 2>/dev/null
```

Common use cases:
- Shared directories where all files should be accessible by a specific group
- Collaborative environments where group ownership needs to be preserved

### Sticky Bit

The sticky bit on a directory prevents users from deleting or renaming files that they don't own, even if they have write permissions on the directory.

```bash
# Set the sticky bit
chmod +t /path/to/directory
# or
chmod 1777 /path/to/directory

# View directories with sticky bit
find / -perm -1000 -type d -exec ls -ld {} \; 2>/dev/null
```

Common use cases:
- `/tmp` directory - All users can create files, but can only delete their own files
- Shared directories where users should not be able to delete each other's files

### Combining Special Permissions

Special permissions can be combined:

```bash
# Set SetUID, SetGID, and sticky bit together
chmod 7755 /path/to/file  # 7 = 4 (SetUID) + 2 (SetGID) + 1 (sticky bit)
```

## Access Control Lists (ACLs)

ACLs provide more fine-grained access control than standard permissions, allowing you to specify permissions for multiple users and groups.

### Prerequisites

```bash
# Install ACL utilities if not already installed
dnf install acl
```

### Checking if ACLs are Enabled

```bash
# Check if a filesystem supports ACLs
tune2fs -l /dev/sda1 | grep "Default mount options"
# or
mount | grep acl
```

### Managing ACLs

```bash
# View ACLs on a file
getfacl file.txt

# Set an ACL for a user
setfacl -m u:username:rwx file.txt

# Set an ACL for a group
setfacl -m g:groupname:rx file.txt

# Set default ACLs on a directory (inherited by new files)
setfacl -d -m u:username:rwx directory/

# Remove a specific ACL
setfacl -x u:username file.txt

# Remove all ACLs
setfacl -b file.txt
```

### ACL Inheritance

```bash
# Set ACLs that will be inherited by new files and directories
setfacl -R -d -m u:username:rwx directory/

# Apply ACLs recursively to existing files and directories
setfacl -R -m u:username:rwx directory/
```

### Backing Up and Restoring ACLs

```bash
# Save ACLs to a file
getfacl -R /path/to/directory > acls.txt

# Restore ACLs from a file
setfacl --restore=acls.txt
```

## Extended Attributes

Extended attributes allow storing additional metadata on files and directories.

```bash
# View extended attributes
getfattr -d file.txt

# Set an extended attribute
setfattr -n user.description -v "Important file" file.txt

# Remove an extended attribute
setfattr -x user.description file.txt
```

## Linux Capabilities

Capabilities provide a more fine-grained breakdown of the privileges traditionally associated with the root user.

### Viewing Capabilities

```bash
# View capabilities of a file
getcap /path/to/file

# View all files with capabilities
find / -type f -exec getcap {} \; 2>/dev/null
```

### Setting Capabilities

```bash
# Set capabilities on a file
setcap cap_net_raw+ep /path/to/program

# Remove capabilities
setcap -r /path/to/program
```

### Common Capabilities

- `cap_net_raw`: Allows use of raw sockets
- `cap_net_bind_service`: Allows binding to ports below 1024
- `cap_sys_admin`: Allows performing system administration tasks
- `cap_dac_override`: Allows bypassing file permission checks
- `cap_setuid`: Allows changing user ID

## Mandatory Access Control (MAC)

Linux supports several MAC systems that provide additional security beyond traditional discretionary access control (DAC).

### SELinux

SELinux (Security-Enhanced Linux) provides mandatory access controls based on security policies.

```bash
# Check SELinux status
getenforce

# View SELinux context of a file
ls -Z file.txt

# Change SELinux context
chcon -t httpd_sys_content_t file.txt

# Restore default SELinux context
restorecon -v file.txt
```

### AppArmor

AppArmor is another MAC system used in some Linux distributions (particularly Ubuntu).

```bash
# Check AppArmor status
aa-status

# View AppArmor profile for a process
aa-unconfined --paranoid

# Put a profile in enforce mode
aa-enforce /etc/apparmor.d/profile.name
```

## Umask

The umask (user mask) determines the default permissions for newly created files and directories.

```bash
# View current umask
umask

# Set umask for the current session
umask 022  # Files: 644, Directories: 755

# Set more restrictive umask
umask 027  # Files: 640, Directories: 750
```

### Calculating Permissions from Umask

- For files: 666 - umask = default permissions
- For directories: 777 - umask = default permissions

### Setting Default Umask

To set the default umask for all users, edit one of these files:
- `/etc/profile`
- `/etc/bashrc`
- `/etc/login.defs`

## Immutable and Append-Only Attributes

The `chattr` command can set special attributes on files that override standard permissions.

```bash
# Make a file immutable (cannot be modified, deleted, or renamed)
chattr +i file.txt

# Make a file append-only (can only be appended to)
chattr +a file.txt

# Remove special attributes
chattr -i file.txt
chattr -a file.txt

# View special attributes
lsattr file.txt
```

## Sudo and Privileged Access

The sudo system allows controlled access to commands with elevated privileges.

### Configuring Sudo Access

Edit the sudoers file with `visudo`:

```bash
# Allow a user to run all commands
username ALL=(ALL) ALL

# Allow a user to run specific commands without a password
username ALL=(ALL) NOPASSWD: /bin/ls, /usr/bin/apt

# Allow a group to run all commands
%wheel ALL=(ALL) ALL
```

### Sudo Command Options

```bash
# Run a command as root
sudo command

# Run a command as another user
sudo -u username command

# Edit a file with elevated privileges
sudo -e /etc/config_file

# Preserve user environment variables
sudo -E command
```

## Polkit (PolicyKit)

Polkit provides a framework for controlling system-wide privileges in a fine-grained way.

```bash
# View Polkit actions
pkaction

# View details of a specific action
pkaction --verbose --action-id org.freedesktop.udisks2.filesystem-mount

# Check if a user is authorized for an action
pkcheck --action-id org.freedesktop.udisks2.filesystem-mount --process $$
```

## Best Practices for Permission Management

1. **Follow the Principle of Least Privilege**: Grant only the permissions necessary for users to perform their tasks.

2. **Use Groups Effectively**: Create groups for different access levels and add users to appropriate groups.

3. **Audit Permissions Regularly**: Periodically review permissions, especially for sensitive files and directories.

4. **Document Special Permissions**: Keep track of files with special permissions and their purpose.

5. **Be Cautious with SetUID/SetGID**: Minimize the use of SetUID and SetGID bits, especially on scripts.

6. **Use ACLs for Complex Requirements**: When standard permissions are not sufficient, use ACLs for fine-grained control.

7. **Implement Mandatory Access Control**: Consider using SELinux or AppArmor for additional security.

8. **Set Appropriate Umask**: Configure a secure default umask for users.

9. **Secure Temporary Directories**: Use the sticky bit on shared directories like `/tmp`.

10. **Use Sudo Instead of Root Login**: Configure sudo access for administrative tasks instead of logging in as root.

## Troubleshooting Permission Issues

### Common Permission Problems

1. **Permission Denied Errors**:
   ```bash
   # Check file permissions
   ls -la file.txt
   
   # Check directory permissions (need execute permission to access files)
   ls -la /path/to/
   
   # Check if SELinux is blocking access
   ausearch -m avc -ts recent
   ```

2. **Cannot Delete a File**:
   ```bash
   # Check if you own the file
   ls -la file.txt
   
   # Check directory permissions (need write permission on directory)
   ls -la /path/to/
   
   # Check for immutable attribute
   lsattr file.txt
   ```

3. **SetUID/SetGID Not Working**:
   ```bash
   # Check if filesystem is mounted with nosuid option
   mount | grep nosuid
   
   # Check if file is on a network filesystem (NFS)
   df -T file.txt
   ```

### Permission Diagnostic Commands

```bash
# Find files with specific permissions
find /path -perm 777 -type f

# Find files owned by a specific user
find /path -user username

# Find files with ACLs
find /path -acl

# Check effective permissions for a user
su - username -c "ls -la /path/to/file"

# Trace permission checks
strace -e open,access,permission_denied command
```

## Advanced Permission Scenarios

### Collaborative Directories

Create a shared directory where multiple users can collaborate:

```bash
# Create directory with SetGID and sticky bit
mkdir /shared
chmod 2775 /shared  # SetGID
chmod +t /shared    # Sticky bit
chown root:project /shared

# Set default ACLs
setfacl -d -m g:project:rwx /shared
```

### Web Server Permissions

Secure permissions for a web server:

```bash
# Set ownership
chown -R root:www-data /var/www/html

# Set permissions
find /var/www/html -type d -exec chmod 750 {} \;
find /var/www/html -type f -exec chmod 640 {} \;

# Set SELinux context
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html
```

### Database File Permissions

Secure permissions for database files:

```bash
# Set ownership
chown -R mysql:mysql /var/lib/mysql

# Set permissions
chmod 700 /var/lib/mysql
chmod 600 /var/lib/mysql/mysql.sock

# Set SELinux context
semanage fcontext -a -t mysqld_db_t "/var/lib/mysql(/.*)?"
restorecon -Rv /var/lib/mysql
```