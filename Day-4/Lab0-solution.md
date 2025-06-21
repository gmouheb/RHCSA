# Lab 0: Repository Configuration - Solution

This document provides the solution for configuring the AppStream and BaseOS repositories in Lab 0.

## Solution

To configure the repositories, you need to create two separate repository files in the `/etc/yum.repos.d/` directory.

### Step 1: Create the BaseOS Repository File

```bash
# Create and edit the BaseOS repository file
sudo vim /etc/yum.repos.d/BaseOS.repo
```

Add the following content to the file:

```ini
[BaseOS]
name=BaseOS
baseurl=https://centos.anexia.at/centos-stream/9-stream/BaseOS/x86_64/os/
gpgcheck=0
enabled=1
```

Save and exit the file (in vim: press `Esc`, then type `:wq` and press `Enter`).

### Step 2: Create the AppStream Repository File

```bash
# Create and edit the AppStream repository file
sudo vim /etc/yum.repos.d/AppStream.repo
```

Add the following content to the file:

```ini
[AppStream]
name=AppStream
baseurl=http://centos.anexia.at/centos-stream/9-stream/AppStream/x86_64/os/
gpgcheck=0
enabled=1
```

Save and exit the file.

### Step 3: Verify the Repository Configuration

After creating both repository files, verify that they are properly configured:

```bash
# Clear the repository cache
sudo dnf clean all

# List all configured repositories
sudo dnf repolist

# Test that you can access packages from both repositories
sudo dnf list available | grep kernel
```

## Notes

- The `gpgcheck=0` setting disables GPG signature verification, which is not recommended for production environments. In a real-world scenario, you should enable GPG checking and import the appropriate GPG keys.
- Notice that we've used `baseurl` instead of `url` in the configuration files, as this is the correct parameter name for DNF/YUM repository files.
- For security reasons, it's generally better to use HTTPS URLs when available.