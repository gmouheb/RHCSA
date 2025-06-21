# Managing Software in Red Hat Linux

This document covers software management in Red Hat Enterprise Linux systems.

## Package Management with YUM/DNF

YUM (Yellowdog Updater Modified) and its successor DNF (Dandified YUM) are the primary package management tools in Red Hat-based systems.

### Basic Package Operations

```bash
# Install a package
dnf install package_name

# Remove a package
dnf remove package_name

# Update a specific package
dnf update package_name

# Update all packages
dnf update

# Search for a package
dnf search keyword
```

### Package Information

```bash
# List all installed packages
dnf list installed

# List all available packages
dnf list available

# Get information about a package
dnf info package_name

# List dependencies for a package
dnf deplist package_name
```

## Repository Management

### Viewing Repositories

```bash
# List all configured repositories
dnf repolist all

# List enabled repositories
dnf repolist enabled

# List disabled repositories
dnf repolist disabled
```

### Managing Repositories

Repository configuration files are stored in `/etc/yum.repos.d/` directory.

```bash
# Enable a repository
dnf config-manager --enable repository_name

# Disable a repository
dnf config-manager --disable repository_name

# Add a repository
dnf config-manager --add-repo repository_url
```

## RPM Package Management

RPM (Red Hat Package Manager) is the underlying package format used in Red Hat systems.

```bash
# Install an RPM package file
rpm -ivh package_file.rpm

# Upgrade an RPM package file
rpm -Uvh package_file.rpm

# Remove an RPM package
rpm -e package_name

# Query information about an installed package
rpm -qi package_name

# List files in an installed package
rpm -ql package_name
```

## Managing Package Groups

```bash
# List available package groups
dnf group list

# Install a package group
dnf group install "group_name"

# Remove a package group
dnf group remove "group_name"
```

## Managing Software Updates

```bash
# Check for available updates
dnf check-update

# Download updates without installing
dnf download package_name

# Apply security updates only
dnf update --security
```

## Subscription Management

Red Hat Enterprise Linux uses subscription-based access to repositories.

```bash
# Register system with Red Hat Subscription Management
subscription-manager register

# List available subscriptions
subscription-manager list --available

# Attach a subscription
subscription-manager attach --auto

# View current subscription status
subscription-manager status
```

## Module Streams (AppStream)

RHEL 8 introduced the concept of module streams for managing multiple versions of software.

```bash
# List available modules
dnf module list

# Enable a module stream
dnf module enable module_name:stream_name

# Install a module
dnf module install module_name:stream_name

# Switch to a different stream
dnf module switch-to module_name:stream_name
```