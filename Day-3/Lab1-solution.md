# Lab 1: Network Configuration - Solution

This document provides solutions for the network configuration tasks in Lab 1.

## Solution

### Task 1: Set the hostname

You can set the hostname using the `nmtui` text-based interface or directly with the `hostnamectl` command:

```bash
# Method 1: Using nmtui (Text User Interface)
nmtui
# Select "Set system hostname"
# Enter: rhcsaserver.example.com
# Select OK

# Method 2: Using hostnamectl (Command Line)
hostnamectl set-hostname rhcsaserver.example.com
```

### Task 2 & 3: Configure IP addresses

Configure a fixed IP address and add a second IP address using `nmtui`:

```bash
# Start the network manager text user interface
nmtui

# Select "Edit a connection"
# Select your network interface (e.g., eth0, ens33)
# Change IPv4 configuration from "Automatic" to "Manual"
# Add your primary IP address with CIDR notation (e.g., 192.168.1.100/24)
# Add gateway address
# Add DNS server (typically same as gateway)
# Add the second IP address: 10.0.0.10/24
# Select OK, then Back
```

Alternatively, you can use the command line:

```bash
# Create a new connection profile with your primary IP
nmcli con add con-name "static-connection" ifname eth0 type ethernet ip4 192.168.1.100/24 gw4 192.168.1.1

# Add the second IP address
nmcli con mod "static-connection" +ipv4.addresses 10.0.0.10/24

# Set DNS server
nmcli con mod "static-connection" ipv4.dns "192.168.1.1"

# Activate the connection
nmcli con up "static-connection"
```

### Task 4: Enable hostname resolution

Add an entry to the `/etc/hosts` file:

```bash
# Edit the hosts file
vim /etc/hosts

# Add the following line (replace YOUR_IP with your actual IP address)
YOUR_IP  rhcsaserver.example.com rhcsaserver
```

### Task 5: Reboot and verify

```bash
# Reboot the system
reboot

# After reboot, verify the configuration
hostname
ip a
ping -c 3 rhcsaserver.example.com
```

## Verification

To verify your network configuration is working correctly:

```bash
# Check hostname
hostname

# Check IP addresses
ip addr show

# Check DNS resolution
getent hosts rhcsaserver.example.com

# Test connectivity
ping -c 3 8.8.8.8
```

If all commands return expected results, your network configuration is working correctly.