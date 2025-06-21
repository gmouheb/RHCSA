# Networking in Linux

This document covers networking concepts and commands in Linux systems.

## Basic Networking Concepts

- IP addressing
- Subnetting
- DNS resolution
- Network interfaces

## Network Configuration

### Viewing Network Information

```bash
# Display all network interfaces and IP addresses
ip addr show

# Display routing table
ip route show

# Check DNS resolution
cat /etc/resolv.conf
```

### Configuring Network Interfaces

```bash
# Configure IP address (temporary)
ip addr add 192.168.1.100/24 dev eth0

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down
```

### Persistent Network Configuration

In Red Hat-based systems, network configuration files are located in:
- `/etc/sysconfig/network-scripts/`

## Network Troubleshooting

### Connectivity Testing

```bash
# Test connectivity to a host
ping example.com

# Trace route to a host
traceroute example.com

# Check listening ports
ss -tuln
```

### DNS Troubleshooting

```bash
# Query DNS records
dig example.com

# Look up hostname
host example.com

# Check DNS resolution for a specific hostname
getent hosts example.com
```

## Network Services

- DHCP client configuration
- Network Manager service
- Firewall configuration

## Advanced Topics

- Network bonding
- Network teaming
- VLANs
- Network bridges