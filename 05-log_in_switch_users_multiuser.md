# Logging In and Switching Users in Multiuser Environments

This guide covers essential operations for working in multiuser Linux environments, including logging in, switching between users, and managing user sessions.

## 1. Logging into a Multiuser System

In a multiuser system like Linux, multiple users can be logged in simultaneously, either locally or remotely.

### Local Login
- To log in locally (from a physical machine), simply enter your username and password when prompted at the login screen or terminal.

### Remote Login
- For remote login, you can use SSH (Secure Shell):

```bash
ssh username@hostname_or_ip_address
```

- You will be prompted to enter the password for the user.
- For enhanced security, you can use SSH key-based authentication instead of passwords.

## 2. Switching Users with su (Switch User)

The `su` command allows you to switch from one user account to another without logging out. This is useful in multiuser environments.

### Basic Syntax
```bash
su - username
```

- The `-` option (often referred to as a login shell) ensures the environment of the new user is loaded, including the home directory, PATH variables, etc.
- Without the `-`, you switch to the user but keep your current environment variables.

### Example
```bash
su - john
```

- You will need to enter the password for the user you are switching to.

## 3. Switching to Root User (Superuser)

To switch to the root user (administrator), you can use:

```bash
su -
```

- You will need to enter the root user's password. 
- This grants you superuser privileges, allowing you to perform administrative tasks.
- Be cautious when operating as root, as you can make system-wide changes that could potentially damage the system.

## 4. Switching Users with sudo

The `sudo` command allows you to execute a command as another user (most commonly as root) without fully switching users.

### Basic Syntax
```bash
sudo -u username command
```

### Examples

Running a command as user john:
```bash
sudo -u john ls /home/john
```
- This command executes `ls` in john's home directory as user john.
- You will need to provide your password (not john's).

Running a command as root (most common usage):
```bash
sudo apt update
```
- This runs the command with root privileges.
- Users must be in the sudoers file to use sudo.

## 5. Returning to the Previous User

After using `su` to switch users, you can return to the previous user by typing:

```bash
exit
```

- This closes the current shell session and switches back to the original user.
- You can also use `Ctrl+D` as a shortcut for `exit`.

## 6. Listing Logged-In Users

To see a list of all users currently logged into the system, use the following commands:

### Using who
```bash
who
```
- This shows the usernames, terminal numbers, and login times for all active sessions.

### Using w
```bash
w
```
- This provides more detailed information, including what each user is currently doing.

### Using last
```bash
last
```
- This shows a history of user logins.

## 7. Switching User Environments (Multiuser Target Mode)

Multiuser mode in Linux allows multiple users to access the system simultaneously, typically with networking enabled.

### Changing System Targets
To switch to a multiuser target (text mode) in systemd-based systems, you can use:

```bash
sudo systemctl isolate multi-user.target
```

- This command will bring the system into a multiuser mode without a graphical interface, allowing multiple users to log in via the terminal.

To switch back to graphical mode:

```bash
sudo systemctl isolate graphical.target
```

### Setting Default Target
To make multi-user target the default boot target:

```bash
sudo systemctl set-default multi-user.target
```

To revert to graphical target as default:

```bash
sudo systemctl set-default graphical.target
```

## Security Considerations

- Always use strong passwords for all user accounts
- Consider implementing SSH key-based authentication for remote logins
- Limit sudo access to only trusted users who need administrative privileges
- Regularly audit user accounts and remove those that are no longer needed
- Monitor login attempts using tools like `lastb` to check for failed login attempts