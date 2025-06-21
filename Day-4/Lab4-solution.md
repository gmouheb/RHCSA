# Lab 4: Systemd Service Configuration - Solution

This document provides solutions for the systemd service configuration tasks in Lab 4.

## Solution

### Task 1: Install Services

Install the required services using dnf:

```bash
sudo dnf install httpd vsftpd -y
```

### Task 2: Set Default Editor

Set vim as the default editor for systemctl:

```bash
export EDITOR=/usr/bin/vim
```

To make this setting permanent, add it to your shell profile:

```bash
echo "export EDITOR=/usr/bin/vim" >> ~/.bashrc
```

### Task 3 & 4: Edit httpd Service Unit File

Use the systemctl edit command to create an override file for the httpd service:

```bash
sudo systemctl edit httpd.service
```

Add the following configuration to the editor:

```ini
[Unit]
Requires=vsftpd.service
After=vsftpd.service

[Service]
Restart=on-failure
RestartSec=10s
```

Save and exit the editor.

The changes you've made:
- `Requires=vsftpd.service`: Ensures that vsftpd is started when httpd starts
- `After=vsftpd.service`: Ensures that vsftpd is started before httpd
- `Restart=on-failure`: Configures the service to restart automatically if it fails
- `RestartSec=10s`: Sets the restart delay to 10 seconds

### Apply and Verify the Changes

Reload the systemd configuration:

```bash
sudo systemctl daemon-reload
```

Test the dependency configuration:

```bash
# First, make sure both services are stopped
sudo systemctl stop httpd
sudo systemctl stop vsftpd

# Verify both services are stopped
sudo systemctl status vsftpd
sudo systemctl status httpd

# Now start only httpd
sudo systemctl start httpd

# Check if vsftpd was automatically started
sudo systemctl status vsftpd
```

You should see that vsftpd is active even though you only started httpd.

### Test the Automatic Restart

To test the automatic restart functionality, you would need to cause the httpd service to fail. One way to do this is to kill the process:

```bash
# Find the httpd process ID
ps aux | grep httpd

# Kill the main httpd process (replace PID with the actual process ID)
sudo kill -9 PID

# Wait 10 seconds and check if the service restarted
sleep 10
sudo systemctl status httpd
```

## Understanding Systemd Unit File Directives

### Unit Section Directives

- **Requires**: Configures requirement dependencies on other units. If a unit has a Requires dependency on another unit, starting the former will result in the latter being started as well.
- **After**: Configures ordering dependencies between units. It ensures that the configured unit is started after the listed unit, but does not cause the other unit to be started.

### Service Section Directives

- **Restart**: Configures whether the service shall be restarted when the service process exits, is killed, or times out.
- **RestartSec**: Configures the time to sleep before restarting a service.

## Additional Information

The changes made using `systemctl edit` are stored in the `/etc/systemd/system/httpd.service.d/override.conf` file. You can view the complete unit file with all overrides using:

```bash
systemctl cat httpd.service
```