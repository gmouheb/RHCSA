# Network Security in Linux

This document covers network security concepts, tools, and best practices for Linux systems.

## Introduction to Network Security

Network security in Linux involves protecting systems from unauthorized access, misuse, modification, or denial of network-accessible resources. Key aspects include:

- Firewall configuration
- Intrusion detection and prevention
- Secure network services
- Network monitoring
- Encryption of network traffic
- Access control

## Firewall Management with firewalld

Firewalld is the default firewall management tool in RHEL/CentOS systems.

### Basic firewalld Concepts

- **Zones**: Pre-defined sets of rules (public, home, work, etc.)
- **Services**: Pre-defined collections of ports and protocols
- **Ports**: Individual port definitions
- **Rich Rules**: Complex rules with more conditions

### Managing firewalld

```bash
# Check firewalld status
systemctl status firewalld

# Start and enable firewalld
systemctl enable --now firewalld

# View current configuration
firewall-cmd --list-all

# View all zones
firewall-cmd --list-all-zones

# Get default zone
firewall-cmd --get-default-zone

# Set default zone
firewall-cmd --set-default-zone=home
```

### Managing Services

```bash
# List available services
firewall-cmd --get-services

# List services in a zone
firewall-cmd --zone=public --list-services

# Add a service to a zone (temporary)
firewall-cmd --zone=public --add-service=http

# Add a service permanently
firewall-cmd --zone=public --add-service=https --permanent

# Remove a service
firewall-cmd --zone=public --remove-service=http --permanent

# Reload firewall to apply changes
firewall-cmd --reload
```

### Managing Ports

```bash
# Add a port to a zone
firewall-cmd --zone=public --add-port=8080/tcp --permanent

# Remove a port
firewall-cmd --zone=public --remove-port=8080/tcp --permanent

# List open ports in a zone
firewall-cmd --zone=public --list-ports
```

### Rich Rules

```bash
# Allow traffic from a specific IP address
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.10" accept' --permanent

# Block traffic from a specific IP address
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.20" drop' --permanent

# Allow a specific IP to access a specific service
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.10" service name="http" accept' --permanent

# Rate limit connections
firewall-cmd --zone=public --add-rich-rule='rule service name="ssh" limit value="3/m" accept' --permanent
```

### Port Forwarding

```bash
# Forward external port 80 to internal port 8080
firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toport=8080 --permanent

# Forward to another IP
firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.10 --permanent

# Forward to another IP and port
firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.1.10 --permanent
```

## Firewall Management with iptables

While firewalld is the preferred tool, iptables is still used in many environments.

### Basic iptables Commands

```bash
# List current rules
iptables -L -v

# Allow SSH traffic
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Block all other incoming traffic
iptables -A INPUT -j DROP

# Save rules (RHEL/CentOS)
service iptables save
```

### Creating a Basic Firewall with iptables

```bash
# Flush existing rules
iptables -F

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback traffic
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
service iptables save
```

## Intrusion Detection with Fail2ban

Fail2ban is a tool that helps protect servers from brute force attacks.

### Installing and Configuring Fail2ban

```bash
# Install Fail2ban
dnf install fail2ban

# Start and enable Fail2ban
systemctl enable --now fail2ban

# Check status
fail2ban-client status
```

### Basic Configuration

The main configuration file is `/etc/fail2ban/jail.conf`, but you should create `/etc/fail2ban/jail.local` for custom settings:

```ini
[DEFAULT]
# Ban hosts for one hour
bantime = 3600
# Check for new log messages every 10 seconds
findtime = 600
# Ban after 5 failures
maxretry = 5

# SSH jail
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/secure
maxretry = 3
```

### Managing Fail2ban

```bash
# Reload configuration
fail2ban-client reload

# Check status of a specific jail
fail2ban-client status sshd

# Manually ban an IP
fail2ban-client set sshd banip 192.168.1.100

# Unban an IP
fail2ban-client set sshd unbanip 192.168.1.100
```

## Secure Shell (SSH) Hardening

SSH is a common target for attacks. Here are ways to secure it:

### SSH Server Configuration

Edit `/etc/ssh/sshd_config`:

```
# Use protocol 2 only
Protocol 2

# Disable root login
PermitRootLogin no

# Use key authentication only
PasswordAuthentication no
ChallengeResponseAuthentication no

# Limit user access
AllowUsers user1 user2

# Change default port (optional)
Port 2222

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding if not needed
X11Forwarding no
```

After making changes:
```bash
# Check configuration syntax
sshd -t

# Restart SSH service
systemctl restart sshd
```

### SSH Key Management

```bash
# Generate SSH key pair
ssh-keygen -t ed25519

# Copy public key to server
ssh-copy-id user@server

# Secure private key permissions
chmod 600 ~/.ssh/id_ed25519
```

## Network Monitoring and Intrusion Detection

### Using Netstat and SS

```bash
# List all listening ports
ss -tulpn

# List established connections
ss -ta

# List all TCP connections
netstat -ant

# List all UDP connections
netstat -anu
```

### Using tcpdump for Network Analysis

```bash
# Capture packets on interface eth0
tcpdump -i eth0

# Capture packets with specific port
tcpdump -i eth0 port 80

# Capture packets from/to specific host
tcpdump -i eth0 host 192.168.1.10

# Write capture to file
tcpdump -i eth0 -w capture.pcap

# Read from capture file
tcpdump -r capture.pcap
```

### Using Wireshark

Wireshark provides a graphical interface for packet analysis:

```bash
# Install Wireshark
dnf install wireshark

# Capture packets (requires root)
wireshark
```

### Intrusion Detection with AIDE

AIDE (Advanced Intrusion Detection Environment) monitors file changes:

```bash
# Install AIDE
dnf install aide

# Initialize database
aide --init

# Copy the database to the active location
cp /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

# Check for changes
aide --check
```

## Securing Network Services

### General Principles

1. **Disable unnecessary services**
2. **Use encrypted protocols** (HTTPS instead of HTTP, SSH instead of Telnet)
3. **Implement access controls**
4. **Keep services updated**
5. **Use strong authentication**

### Securing Web Servers (Apache/Nginx)

```bash
# Install mod_ssl for Apache
dnf install mod_ssl

# Generate SSL certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/private/server.key -out /etc/pki/tls/certs/server.crt

# Configure SSL in Apache
# Edit /etc/httpd/conf.d/ssl.conf
```

Security headers for Apache:
```apache
# Add to Apache configuration
Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-XSS-Protection "1; mode=block"
Header always set Content-Security-Policy "default-src 'self'"
```

### Securing Database Servers

```bash
# MySQL/MariaDB security script
mysql_secure_installation

# PostgreSQL - Edit pg_hba.conf for access control
# Example: Allow only local connections
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

### Securing Mail Servers

```bash
# Postfix configuration for TLS
# Edit /etc/postfix/main.cf
smtpd_tls_cert_file = /etc/pki/tls/certs/server.crt
smtpd_tls_key_file = /etc/pki/tls/private/server.key
smtpd_tls_security_level = may
smtp_tls_security_level = may
```

## Network Encryption

### Using OpenVPN

```bash
# Install OpenVPN
dnf install openvpn

# Generate server configuration
# Example configuration in /etc/openvpn/server.conf
```

Basic server configuration:
```
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
comp-lzo
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
verb 3
```

### Using WireGuard

```bash
# Install WireGuard
dnf install wireguard-tools

# Generate private and public keys
wg genkey | tee privatekey | wg pubkey > publickey

# Create configuration
# Example /etc/wireguard/wg0.conf
```

Basic WireGuard configuration:
```
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
```

## Security Scanning and Auditing

### Using Nmap for Network Scanning

```bash
# Install Nmap
dnf install nmap

# Basic scan
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 192.168.1.10

# Service and version detection
nmap -sV 192.168.1.10

# OS detection
nmap -O 192.168.1.10

# Comprehensive scan
nmap -A 192.168.1.10
```

### Using OpenVAS for Vulnerability Scanning

```bash
# Install OpenVAS (may require additional repositories)
dnf install openvas

# Setup OpenVAS
openvas-setup

# Access web interface
# https://localhost:9392
```

### Using Lynis for Security Auditing

```bash
# Install Lynis
dnf install lynis

# Run system audit
lynis audit system

# View report
cat /var/log/lynis.log
```

## Network Access Control

### Using TCP Wrappers

TCP Wrappers provides host-based access control:

```bash
# Allow SSH access from specific network
echo "sshd: 192.168.1.0/24" >> /etc/hosts.allow

# Deny all other access
echo "sshd: ALL" >> /etc/hosts.deny
```

### Using SELinux for Network Control

```bash
# Allow Apache to connect to network
setsebool -P httpd_can_network_connect on

# Allow Apache to connect to database
setsebool -P httpd_can_network_connect_db on

# Allow service to use non-standard port
semanage port -a -t http_port_t -p tcp 8080
```

## Secure Network Time

Accurate time is crucial for security:

```bash
# Install chrony
dnf install chrony

# Configure NTP servers in /etc/chrony.conf
server 0.pool.ntp.org
server 1.pool.ntp.org
server 2.pool.ntp.org
server 3.pool.ntp.org

# Start and enable chronyd
systemctl enable --now chronyd

# Check time synchronization
chronyc tracking
```

## Best Practices for Network Security

1. **Defense in Depth**: Implement multiple layers of security
2. **Principle of Least Privilege**: Grant only necessary access
3. **Keep Systems Updated**: Apply security patches promptly
4. **Regular Backups**: Maintain and test backups regularly
5. **Network Segmentation**: Separate networks by function
6. **Monitoring and Logging**: Monitor for suspicious activity
7. **Security Policies**: Develop and enforce security policies
8. **Regular Security Audits**: Conduct periodic security assessments
9. **Incident Response Plan**: Prepare for security incidents
10. **User Education**: Train users on security best practices

## Troubleshooting Network Security Issues

### Common Issues and Solutions

1. **Firewall Blocking Legitimate Traffic**:
   ```bash
   # Temporarily disable firewall to test
   systemctl stop firewalld
   
   # Check if service works, then add appropriate rule
   firewall-cmd --zone=public --add-service=http --permanent
   firewall-cmd --reload
   ```

2. **SSH Connection Issues**:
   ```bash
   # Check SSH service status
   systemctl status sshd
   
   # Check SSH configuration
   sshd -t
   
   # Check firewall
   firewall-cmd --list-all
   ```

3. **SELinux Blocking Network Access**:
   ```bash
   # Check SELinux status
   getenforce
   
   # View SELinux denials
   ausearch -m AVC -ts recent
   
   # Allow specific access
   setsebool -P httpd_can_network_connect on
   ```

4. **Fail2ban Blocking Legitimate Access**:
   ```bash
   # Check banned IPs
   fail2ban-client status sshd
   
   # Unban an IP
   fail2ban-client set sshd unbanip 192.168.1.10
   ```

### Diagnostic Tools

```bash
# Check network connectivity
ping host
traceroute host

# Check DNS resolution
dig domain
nslookup domain

# Check port connectivity
telnet host port
nc -zv host port

# Check routing
ip route
route -n
```