# Managing Local User Accounts and Groups

This guide covers essential operations for managing local user accounts, groups, and superuser access in Linux systems.

## 1. Create, Delete, and Modify Local User Accounts

### Creating a New User

Use the `useradd` command to create a new user account:

```bash
sudo useradd username
```

To create a user with a home directory and a default shell:

```bash
sudo useradd -m -s /bin/bash username
```

Common options:
- `-m`: Create the user's home directory
- `-s`: Specify the user's login shell
- `-c`: Add a comment (usually full name)
- `-G`: Add the user to additional groups
- `-d`: Specify a custom home directory location

Example with multiple options:
```bash
sudo useradd -m -s /bin/bash -c "John Doe" -G wheel,developers john
```

### Deleting a User Account

To delete a user and their home directory:

```bash
sudo userdel -r username
```

To delete a user but keep their files:

```bash
sudo userdel username
```

### Modifying a User Account

To modify an existing user, such as changing their home directory or shell:

```bash
# Change home directory
sudo usermod -d /new/home/directory username

# Change shell
sudo usermod -s /bin/zsh username

# Change username
sudo usermod -l new_username old_username

# Change user ID (UID)
sudo usermod -u 1001 username
```

## 2. Change Passwords and Adjust Password Aging for Local User Accounts

### Changing a User Password

To change the password for a user:

```bash
sudo passwd username
```

You will be prompted to enter a new password.

To allow a user to have no password (not recommended for security reasons):

```bash
sudo passwd -d username
```

### Configuring Password Aging

To view password aging information for a user:

```bash
sudo chage -l username
```

To set the password to expire after a certain number of days:

```bash
sudo chage -M 90 username  # Password expires after 90 days
```

To force a user to change their password at the next login:

```bash
sudo chage -d 0 username
```

Other useful options:
- `-m`: Minimum number of days between password changes
- `-W`: Number of days of warning before password expires
- `-I`: Number of days after password expires until account is disabled
- `-E`: Set account expiration date (YYYY-MM-DD)

Example setting multiple aging parameters:
```bash
sudo chage -m 7 -M 90 -W 14 -I 30 username
```

## 3. Create, Delete, and Modify Local Groups and Group Memberships

### Creating a New Group

To create a group:

```bash
sudo groupadd groupname
```

To create a group with a specific GID:

```bash
sudo groupadd -g 1001 groupname
```

### Deleting a Group

To delete a group:

```bash
sudo groupdel groupname
```

### Adding a User to a Group

To add an existing user to a group:

```bash
sudo usermod -aG groupname username
```

The `-a` option is important as it appends the group to the user's existing groups. Without it, the user would be removed from all groups not specified in the command.

To add a user to multiple groups at once:

```bash
sudo usermod -aG group1,group2,group3 username
```

### Removing a User from a Group

To remove a user from a group:

```bash
sudo gpasswd -d username groupname
```

### Viewing Group Memberships

To view the groups a user belongs to:

```bash
groups username
```

To see all groups on the system:

```bash
getent group
```

To see all members of a specific group:

```bash
getent group groupname
```

## 4. Configure Superuser Access

### Granting Superuser (sudo) Access

To grant a user sudo (superuser) privileges, add them to the sudo group (Ubuntu/Debian) or wheel group (RHEL/CentOS):

For Ubuntu/Debian:
```bash
sudo usermod -aG sudo username
```

For RHEL/CentOS:
```bash
sudo usermod -aG wheel username
```

### Modifying sudo Access with /etc/sudoers

The `/etc/sudoers` file controls who can use sudo and which commands they are allowed to run. This file should always be edited using `visudo`, a tool that checks the syntax before saving changes:

```bash
sudo visudo
```

You can add lines like this to allow a user to run all commands with sudo:

```
username ALL=(ALL) ALL
```

### Using /etc/sudoers.d/ for Custom sudo Rules

Instead of modifying `/etc/sudoers` directly, you can place custom sudo rules in separate files under the `/etc/sudoers.d/` directory. This is a safer and more modular approach:

```bash
sudo visudo -f /etc/sudoers.d/custom_file
```

For example, to allow a specific user to run a particular command without a password:

```
username ALL=(ALL) NOPASSWD: /usr/bin/specific_command
```

To block a specific user from running a particular command:

```
username ALL=(ALL) ALL, !/usr/bin/command_to_block
```

These custom files follow the same syntax rules as `/etc/sudoers` and are processed in order.

### Restricting sudo Access

To remove a user's sudo privileges, simply remove them from the sudo or wheel group:

For Ubuntu/Debian:
```bash
sudo deluser username sudo
```

For RHEL/CentOS:
```bash
sudo gpasswd -d username wheel
```

## 5. Important Files for User Management

- `/etc/passwd`: Contains basic user account information
- `/etc/shadow`: Contains encrypted password information
- `/etc/group`: Contains group information
- `/etc/gshadow`: Contains encrypted group passwords
- `/etc/login.defs`: Defines the site-specific configuration for the shadow password suite
- `/etc/skel/`: Contains files that are copied to a new user's home directory

## 6. Best Practices for User Management

1. Use strong passwords and enforce password policies
2. Regularly audit user accounts and remove unnecessary accounts
3. Follow the principle of least privilege when granting permissions
4. Use groups to manage permissions for multiple users
5. Document user account creation and deletion procedures
6. Regularly review sudo access and permissions
7. Consider using centralized authentication for larger environments