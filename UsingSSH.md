# Using SSH in Linux

## Introduction to SSH

SSH (Secure Shell) is a cryptographic network protocol used for secure communication over an unsecured network. It provides a secure channel over an unsecured network by using strong encryption, and is widely used for remote login, remote command execution, and secure file transfers.

## Basic SSH Commands

### Connecting to a Remote Server
```bash
ssh username@hostname
```

Example:
```bash
ssh user@192.168.1.100
ssh admin@server.example.com
```

### Using a Different Port
```bash
ssh -p port_number username@hostname
```

Example:
```bash
ssh -p 2222 user@192.168.1.100
```

### SSH with Key-Based Authentication
```bash
ssh -i /path/to/private_key username@hostname
```

## SSH Key Management

### Generating SSH Keys
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Options:
- `-t`: Type of key (rsa, ed25519, etc.)
- `-b`: Bits in the key (higher is more secure)
- `-C`: Comment (usually email address)

### Copying Your Public Key to a Server
```bash
ssh-copy-id username@hostname
```

Or manually:
```bash
cat ~/.ssh/id_rsa.pub | ssh username@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Managing Known Hosts
SSH stores the fingerprints of servers you connect to in `~/.ssh/known_hosts`. If a server's fingerprint changes, SSH will warn you.

To remove a known host:
```bash
ssh-keygen -R hostname
```

## SSH Configuration

### SSH Client Configuration
Create or edit `~/.ssh/config`:

```
Host myserver
    HostName server.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_rsa_server

Host *
    ServerAliveInterval 60
```

With this configuration, you can simply use:
```bash
ssh myserver
```

### SSH Server Configuration
The SSH server configuration is typically located at `/etc/ssh/sshd_config`. Common settings include:

```
# Disable root login
PermitRootLogin no

# Use only SSH protocol version 2
Protocol 2

# Allow only specific users
AllowUsers user1 user2

# Change default port
Port 2222

# Disable password authentication (use keys only)
PasswordAuthentication no
```

After changing the configuration, restart the SSH service:
```bash
sudo systemctl restart sshd
```

## Secure File Transfer with SSH

### SCP (Secure Copy)
```bash
# Copy local file to remote server
scp /path/to/local/file username@hostname:/path/to/remote/directory

# Copy remote file to local system
scp username@hostname:/path/to/remote/file /path/to/local/directory

# Copy directory recursively
scp -r /path/to/local/directory username@hostname:/path/to/remote/directory
```

### SFTP (SSH File Transfer Protocol)
```bash
# Start SFTP session
sftp username@hostname
```

Common SFTP commands:
- `ls`: List files
- `cd`: Change directory
- `get`: Download file
- `put`: Upload file
- `mkdir`: Create directory
- `rm`: Remove file
- `exit`: Close connection

## SSH Tunneling

### Local Port Forwarding
```bash
ssh -L local_port:destination_host:destination_port username@ssh_server
```

Example (access remote MySQL server):
```bash
ssh -L 3306:localhost:3306 user@database_server
```

### Remote Port Forwarding
```bash
ssh -R remote_port:destination_host:destination_port username@ssh_server
```

### Dynamic Port Forwarding (SOCKS Proxy)
```bash
ssh -D local_port username@ssh_server
```

## SSH Security Best Practices

1. Use key-based authentication instead of passwords
2. Use strong, unique keys (RSA 4096 bits or ED25519)
3. Protect your private keys with passphrases
4. Disable root login
5. Use non-standard ports
6. Implement fail2ban to prevent brute force attacks
7. Keep your SSH client and server updated
8. Use SSH configuration to simplify connections
9. Limit user access with AllowUsers or AllowGroups
10. Consider using Multi-Factor Authentication for critical systems

## Troubleshooting SSH Connections

### Verbose Mode
```bash
ssh -v username@hostname    # Verbose
ssh -vv username@hostname   # More verbose
ssh -vvv username@hostname  # Even more verbose
```

### Common Issues
- Permission problems: Ensure private key permissions are set to 600 (`chmod 600 ~/.ssh/id_rsa`)
- Firewall blocking: Check if port 22 (or your custom port) is open
- SSH service not running: `systemctl status sshd`
- Authentication failures: Check logs with `journalctl -u sshd`