# SELinux (Security-Enhanced Linux)

This document covers SELinux concepts, configuration, and management in Linux systems.

## Introduction to SELinux

Security-Enhanced Linux (SELinux) is a security mechanism implemented in the Linux kernel that provides mandatory access control (MAC) beyond the traditional discretionary access control (DAC) model. SELinux was developed by the NSA and integrated into the Linux kernel to enhance system security.

Key features of SELinux include:

- Mandatory access controls
- Fine-grained permission system
- Type enforcement
- Role-based access control
- Multi-level security

## SELinux Modes

SELinux can operate in three modes:

1. **Enforcing**: SELinux policy is enforced. Security violations are denied and logged.
2. **Permissive**: SELinux policy is not enforced, but violations are logged. Useful for troubleshooting.
3. **Disabled**: SELinux is completely disabled.

### Checking SELinux Mode

```bash
# Check current SELinux mode
getenforce

# Get detailed SELinux status
sestatus
```

### Changing SELinux Mode

```bash
# Temporarily set to permissive mode
setenforce 0

# Temporarily set to enforcing mode
setenforce 1

# Permanently change mode by editing /etc/selinux/config
vi /etc/selinux/config
```

Example `/etc/selinux/config`:
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

## SELinux Contexts

SELinux uses contexts to determine access permissions. A context consists of:

- User
- Role
- Type (most commonly used)
- Level (optional, used in MLS)

### Viewing SELinux Contexts

```bash
# View file contexts
ls -Z /var/www/html/

# View process contexts
ps -eZ | grep httpd

# View user contexts
id -Z
```

### Understanding Context Format

SELinux contexts are displayed in the format:
```
user:role:type:level
```

For example:
```
system_u:object_r:httpd_sys_content_t:s0
```

- `system_u` - SELinux user
- `object_r` - SELinux role
- `httpd_sys_content_t` - SELinux type
- `s0` - SELinux level

## Managing File Contexts

### Changing File Contexts

```bash
# Change the context of a file
chcon -t httpd_sys_content_t file.html

# Change context recursively
chcon -R -t httpd_sys_content_t /var/www/html/

# Change context to match reference file
chcon --reference=/var/www/html/index.html file.html
```

### Restoring Default Contexts

```bash
# Restore default context for a file
restorecon file.html

# Restore default contexts recursively
restorecon -R /var/www/html/

# Restore with verbose output
restorecon -Rv /var/www/html/
```

### Managing Context Rules

```bash
# List file context rules
semanage fcontext -l

# Add a new file context rule
semanage fcontext -a -t httpd_sys_content_t "/custom/web(/.*)?"

# Delete a file context rule
semanage fcontext -d "/custom/web(/.*)"

# Apply the rules (after adding/changing them)
restorecon -Rv /custom/web/
```

## SELinux Booleans

SELinux booleans are switches that change the behavior of the SELinux policy without modifying the policy itself.

### Managing Booleans

```bash
# List all booleans
getsebool -a

# List booleans related to HTTP
getsebool -a | grep http

# Get specific boolean value
getsebool httpd_can_network_connect

# Set boolean temporarily
setsebool httpd_can_network_connect on

# Set boolean permanently
setsebool -P httpd_can_network_connect on
```

### Common SELinux Booleans

- `httpd_can_network_connect` - Allow HTTP server to make network connections
- `httpd_can_network_connect_db` - Allow HTTP server to connect to databases
- `httpd_can_sendmail` - Allow HTTP server to send mail
- `httpd_enable_homedirs` - Allow HTTP server to access user home directories
- `samba_share_nfs` - Allow Samba to share NFS volumes
- `ssh_sysadm_login` - Allow SSH logins by sysadm roles

## SELinux Ports

SELinux controls which ports services can use.

### Managing Port Contexts

```bash
# List all port contexts
semanage port -l

# List HTTP port contexts
semanage port -l | grep http

# Add a new port context
semanage port -a -t http_port_t -p tcp 8080

# Modify an existing port context
semanage port -m -t http_port_t -p tcp 8080

# Delete a port context
semanage port -d -t http_port_t -p tcp 8080
```

## SELinux Users

SELinux has its own user concept separate from Linux users.

### Managing SELinux Users

```bash
# List SELinux users
semanage user -l

# Map Linux user to SELinux user
semanage login -a -s staff_u john

# List Linux user to SELinux user mappings
semanage login -l
```

## SELinux Troubleshooting

### Analyzing SELinux Logs

SELinux logs denials to `/var/log/audit/audit.log` and often to `/var/log/messages`.

```bash
# View SELinux denials in audit log
ausearch -m AVC,USER_AVC -ts recent

# View SELinux denials with more context
ausearch -m AVC,USER_AVC -ts recent | audit2allow -a
```

### Using audit2why and audit2allow

```bash
# Explain why access was denied
ausearch -m AVC -ts recent | audit2why

# Generate policy module to allow denied access
ausearch -m AVC -ts recent | audit2allow -M mymodule

# Install the generated policy module
semodule -i mymodule.pp
```

### Common SELinux Issues and Solutions

#### Web Server Access Issues

```bash
# Allow Apache to access content in non-standard directory
semanage fcontext -a -t httpd_sys_content_t "/custom/web(/.*)"
restorecon -Rv /custom/web/

# Allow Apache to connect to network
setsebool -P httpd_can_network_connect on

# Allow Apache to connect to database
setsebool -P httpd_can_network_connect_db on
```

#### File Transfer Issues

```bash
# Allow SFTP/SCP to access home directories
setsebool -P ssh_sysadm_login on

# Fix context for files uploaded via SFTP/SCP
restorecon -Rv /home/user/
```

#### Custom Service Port Issues

```bash
# Allow service to use non-standard port
semanage port -a -t http_port_t -p tcp 8080
```

## SELinux Policy Modules

### Managing Policy Modules

```bash
# List installed policy modules
semodule -l

# Install a policy module
semodule -i mymodule.pp

# Remove a policy module
semodule -r mymodule

# Disable a policy module
semodule -d mymodule

# Enable a policy module
semodule -e mymodule
```

### Creating Custom Policy Modules

```bash
# Create a policy module from denials
ausearch -m AVC -ts recent | audit2allow -M mymodule

# Edit the policy module (optional)
vi mymodule.te

# Compile the policy module
make -f /usr/share/selinux/devel/Makefile mymodule.pp

# Install the policy module
semodule -i mymodule.pp
```

## SELinux File Labels

### Understanding File Label Components

File labels consist of four components:
- **User**: The SELinux user identity
- **Role**: The SELinux role
- **Type**: The SELinux type
- **Level**: The MLS/MCS security level

### Common File Types

- `httpd_sys_content_t` - Web content
- `httpd_sys_script_exec_t` - Executable web scripts
- `httpd_sys_rw_content_t` - Web content that can be modified by the web server
- `samba_share_t` - Samba shared files
- `public_content_t` - Content accessible by various network services
- `user_home_t` - User home directory content

## SELinux and Services

### Apache (httpd)

```bash
# Allow Apache to connect to the network
setsebool -P httpd_can_network_connect on

# Allow Apache to connect to databases
setsebool -P httpd_can_network_connect_db on

# Allow Apache to access user home directories
setsebool -P httpd_enable_homedirs on
restorecon -Rv /home/*/public_html
```

### Samba

```bash
# Allow Samba to share user home directories
setsebool -P samba_enable_home_dirs on

# Allow Samba to share NFS volumes
setsebool -P samba_share_nfs on

# Set correct context for Samba shares
semanage fcontext -a -t samba_share_t "/srv/samba(/.*)"
restorecon -Rv /srv/samba
```

### Database (MySQL/MariaDB)

```bash
# Allow database to access NFS
setsebool -P mysqld_disable_trans on

# Set correct context for database files
semanage fcontext -a -t mysqld_db_t "/path/to/custom/datadir(/.*)"
restorecon -Rv /path/to/custom/datadir
```

## Advanced SELinux Topics

### Multi-Level Security (MLS)

MLS provides additional security by implementing sensitivity levels and categories.

```bash
# View MLS settings
semanage user -l

# Change MLS range for a file
chcon -l s0:c0.c1023 file.txt
```

### SELinux Policy Development

For developing custom SELinux policies:

```bash
# Install policy development tools
dnf install selinux-policy-devel

# Create a policy template
sepolicy generate --init /usr/local/bin/myapp

# Build and install the policy
cd myapp
make -f /usr/share/selinux/devel/Makefile
semodule -i myapp.pp
```

## Best Practices

1. **Don't disable SELinux**: Use permissive mode for troubleshooting instead of disabling SELinux completely.

2. **Use standard directories**: When possible, use directories with predefined contexts (e.g., `/var/www/html` for web content).

3. **Modify contexts, not the policy**: Use `semanage fcontext` and `setsebool` instead of creating custom policies when possible.

4. **Document changes**: Keep track of all SELinux customizations for future reference.

5. **Use tools, not brute force**: Leverage `audit2allow`, `audit2why`, and `sealert` for troubleshooting instead of blindly changing contexts.

6. **Test in permissive mode**: Before deploying applications in enforcing mode, test them in permissive mode to identify potential issues.

7. **Keep SELinux packages updated**: Ensure SELinux policy packages are up to date to benefit from security improvements.

8. **Use SELinux MLS for high-security environments**: Consider using MLS for environments with strict security requirements.

9. **Regularly review AVC denials**: Monitor for unauthorized access attempts and potential policy issues.

10. **Train administrators**: Ensure system administrators understand SELinux concepts and troubleshooting techniques.