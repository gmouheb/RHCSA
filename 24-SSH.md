# SSH (Secure Shell) in Linux

This document covers SSH concepts, configuration, and usage in Linux systems.

## Introduction to SSH

SSH (Secure Shell) is a cryptographic network protocol for secure communication over an unsecured network. It provides:

- Encrypted communication between systems
- Secure remote login and command execution
- Secure file transfers
- Port forwarding and tunneling
- Key-based authentication

## SSH Components

The SSH protocol consists of several components:

- **SSH Server (sshd)**: Runs on the remote system, accepting connections
- **SSH Client**: Used to connect to SSH servers
- **SSH Keys**: Used for authentication
- **SSH Config Files**: Control behavior of SSH client and server

## Installing SSH

### On Red Hat/CentOS/Fedora

```bash
# Install SSH server and client
dnf install openssh-server openssh-clients

# Start and enable SSH server
systemctl enable --now sshd
```

## Basic SSH Usage

### Connecting to a Remote System

```bash
# Basic SSH connection
ssh username@hostname

# Specify a different port
ssh -p 2222 username@hostname

# Verbose output for troubleshooting
ssh -v username@hostname

# More verbose output
ssh -vv username@hostname
```

### Executing Remote Commands

```bash
# Execute a command on the remote system
ssh username@hostname "ls -la"

# Multiple commands
ssh username@hostname "cd /tmp && ls -la"
```

## SSH Key Authentication

SSH keys provide a more secure alternative to password authentication.

### Generating SSH Keys

```bash
# Generate RSA key pair (default)
ssh-keygen

# Generate Ed25519 key (more secure, recommended)
ssh-keygen -t ed25519

# Generate RSA key with 4096 bits
ssh-keygen -t rsa -b 4096

# Generate key with custom file name
ssh-keygen -t ed25519 -f ~/.ssh/mykey

# Generate key with a comment (useful for identification)
ssh-keygen -t ed25519 -C "work laptop"
```

### Copying SSH Public Key to Remote Server

```bash
# Using ssh-copy-id (recommended)
ssh-copy-id username@hostname

# Using ssh-copy-id with custom key
ssh-copy-id -i ~/.ssh/mykey.pub username@hostname

# Manual method
cat ~/.ssh/id_ed25519.pub | ssh username@hostname "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Using SSH Keys with Different Names

```bash
# Specify key file when connecting
ssh -i ~/.ssh/mykey username@hostname

# Add key to SSH agent
eval $(ssh-agent)
ssh-add ~/.ssh/mykey
```

## SSH Client Configuration

The SSH client configuration file is located at `~/.ssh/config` (user-specific) or `/etc/ssh/ssh_config` (system-wide).

### Basic Configuration Examples

```
# Global settings
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Specific host configuration
Host server1
    HostName server1.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/server1_key

# Wildcard matching
Host *.example.com
    User admin
    IdentityFile ~/.ssh/example_key

# Jump host (bastion) configuration
Host internal
    HostName internal.example.local
    User admin
    ProxyJump bastion.example.com
```

### Common Client Configuration Options

- `HostName`: Actual hostname to connect to
- `User`: Username to use for connection
- `Port`: SSH port number
- `IdentityFile`: Path to private key file
- `ForwardAgent`: Enable/disable SSH agent forwarding
- `ForwardX11`: Enable/disable X11 forwarding
- `ServerAliveInterval`: Time interval for sending keep-alive packets
- `Compression`: Enable/disable compression
- `StrictHostKeyChecking`: Host key verification behavior
- `ProxyJump`: Specify a jump host for connection

## SSH Server Configuration

The SSH server configuration file is located at `/etc/ssh/sshd_config`.

### Basic Server Configuration

```bash
# Edit SSH server configuration
vi /etc/ssh/sshd_config
```

Common settings:
```
# Network settings
Port 22
ListenAddress 0.0.0.0
ListenAddress ::

# Authentication settings
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Security settings
X11Forwarding yes
PermitEmptyPasswords no
MaxAuthTries 3
```

### Applying Configuration Changes

```bash
# Check configuration syntax
sshd -t

# Restart SSH server to apply changes
systemctl restart sshd
```

## SSH Security Best Practices

### Server Security

1. **Disable root login**:
   ```
   PermitRootLogin no
   ```

2. **Use key-based authentication and disable password authentication**:
   ```
   PubkeyAuthentication yes
   PasswordAuthentication no
   ```

3. **Change default port**:
   ```
   Port 2222
   ```

4. **Limit user access**:
   ```
   AllowUsers user1 user2
   AllowGroups sshusers
   ```

5. **Configure idle timeout**:
   ```
   ClientAliveInterval 300
   ClientAliveCountMax 2
   ```

6. **Disable unused features**:
   ```
   X11Forwarding no
   AllowTcpForwarding no
   ```

7. **Use strong ciphers and MACs**:
   ```
   Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
   MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
   ```

### Client Security

1. **Use SSH keys with passphrases**
2. **Keep SSH client software updated**
3. **Verify host keys on first connection**
4. **Use `HashKnownHosts yes` in client configuration**
5. **Use jump hosts for accessing internal systems**

## SSH File Transfer

### Using SCP (Secure Copy)

```bash
# Copy local file to remote system
scp file.txt username@hostname:/path/to/destination/

# Copy remote file to local system
scp username@hostname:/path/to/file.txt local_destination/

# Copy directory recursively
scp -r directory/ username@hostname:/path/to/destination/

# Specify port
scp -P 2222 file.txt username@hostname:/path/to/destination/
```

### Using SFTP (SSH File Transfer Protocol)

```bash
# Start SFTP session
sftp username@hostname

# SFTP commands
# pwd - Print remote working directory
# lpwd - Print local working directory
# ls - List remote files
# lls - List local files
# cd - Change remote directory
# lcd - Change local directory
# get file - Download file
# put file - Upload file
# exit - Exit SFTP
```

## SSH Port Forwarding

SSH port forwarding (tunneling) allows secure access to services through an SSH connection.

### Local Port Forwarding

Forward a local port to a remote destination through an SSH server:

```bash
# Forward local port 8080 to remote server's port 80
ssh -L 8080:remote-server:80 username@ssh-server

# Access the service
curl http://localhost:8080/
```

### Remote Port Forwarding

Forward a remote port to a local destination:

```bash
# Forward remote port 8080 to local port 80
ssh -R 8080:localhost:80 username@ssh-server
```

### Dynamic Port Forwarding (SOCKS Proxy)

Create a SOCKS proxy for flexible forwarding:

```bash
# Create SOCKS proxy on local port 1080
ssh -D 1080 username@ssh-server

# Use with applications that support SOCKS proxies
# For example, configure browser to use SOCKS proxy localhost:1080
```

## SSH Jump Hosts

Jump hosts (bastion hosts) provide a secure way to access internal systems.

```bash
# Connect through a jump host
ssh -J jumpuser@jumphost targetuser@targethost

# In SSH config file
Host target
    HostName targethost
    User targetuser
    ProxyJump jumpuser@jumphost
```

## SSH Agent

SSH agent holds private keys in memory for authentication.

```bash
# Start SSH agent
eval $(ssh-agent)

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Add key with custom lifetime (seconds)
ssh-add -t 3600 ~/.ssh/id_ed25519

# List keys in agent
ssh-add -l

# Remove specific key
ssh-add -d ~/.ssh/id_ed25519

# Remove all keys
ssh-add -D
```

## SSH Agent Forwarding

Agent forwarding allows using local SSH keys on remote systems.

```bash
# Connect with agent forwarding
ssh -A username@hostname

# In SSH config file
Host server
    HostName server.example.com
    User admin
    ForwardAgent yes
```

**Security note**: Only use agent forwarding with trusted servers.

## X11 Forwarding

X11 forwarding allows running graphical applications on a remote system.

```bash
# Connect with X11 forwarding
ssh -X username@hostname

# More secure X11 forwarding
ssh -Y username@hostname

# Run graphical application
firefox
```

## SSH Multiplexing

SSH multiplexing reuses connections for faster subsequent connections.

```bash
# In SSH config file
Host *
    ControlMaster auto
    ControlPath ~/.ssh/control:%h:%p:%r
    ControlPersist 1h
```

## SSH Escape Sequences

During an SSH session, you can use escape sequences by typing `~` followed by a character:

- `~.` - Terminate connection
- `~^Z` - Suspend SSH connection
- `~#` - List forwarded connections
- `~?` - Display help text

## Troubleshooting SSH

### Common Issues and Solutions

1. **Connection refused**:
   - Check if SSH server is running: `systemctl status sshd`
   - Check firewall settings: `firewall-cmd --list-all`
   - Verify SSH port: `ss -tulpn | grep ssh`

2. **Permission denied**:
   - Check username and password
   - Verify SSH key permissions (should be 600 for private keys)
   - Check authorized_keys file permissions (should be 600)
   - Check ~/.ssh directory permissions (should be 700)

3. **Host key verification failed**:
   - Remove old host key: `ssh-keygen -R hostname`
   - Manually verify and add host key

4. **Slow SSH connection**:
   - Disable DNS lookups in server config: `UseDNS no`
   - Use compression: `Compression yes`
   - Check for reverse DNS issues

### Debugging SSH Connections

```bash
# Client-side debugging
ssh -v username@hostname    # Verbose
ssh -vv username@hostname   # More verbose
ssh -vvv username@hostname  # Even more verbose

# Server-side debugging
# Edit /etc/ssh/sshd_config and set:
LogLevel DEBUG3
# Then restart sshd and check logs:
journalctl -u sshd
```

## Advanced SSH Topics

### SSH Certificates

SSH certificates provide a more scalable way to manage SSH authentication.

```bash
# Create a Certificate Authority (CA) key
ssh-keygen -t ed25519 -f ca_key

# Sign a user's public key
ssh-keygen -s ca_key -I user_id -n username -V +52w id_ed25519.pub

# Configure server to trust the CA
# In /etc/ssh/sshd_config:
TrustedUserCAKeys /etc/ssh/ca.pub
```

### SSH Hardening with PAM

Pluggable Authentication Modules (PAM) can enhance SSH security.

```bash
# Configure two-factor authentication with Google Authenticator
dnf install google-authenticator
# Edit /etc/pam.d/sshd and add:
auth required pam_google_authenticator.so
# Edit /etc/ssh/sshd_config:
ChallengeResponseAuthentication yes
```

### SSH Auditing and Logging

```bash
# Configure detailed logging
# In /etc/ssh/sshd_config:
LogLevel VERBOSE
# Check logs:
journalctl -u sshd

# Use auditd for SSH auditing
dnf install audit
auditctl -w /etc/ssh/ -p wa -k sshd_config
```

## SSH Tools and Utilities

- **ssh-copy-id**: Installs SSH keys on a server
- **ssh-keygen**: Generates, manages, and converts SSH keys
- **ssh-agent**: Holds private keys for authentication
- **ssh-add**: Adds keys to the SSH agent
- **sshpass**: Non-interactive SSH password provider (use with caution)
- **sshfs**: Mount remote filesystems over SSH
- **autossh**: Automatically restart SSH sessions and tunnels