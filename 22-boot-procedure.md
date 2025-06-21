# Linux Boot Procedure

This document covers the boot process in Linux systems, with a focus on RHEL/CentOS systems.

## Overview of the Linux Boot Process

The Linux boot process consists of several stages:

1. **Power-On Self-Test (POST)**
2. **Boot Loader (GRUB2)**
3. **Kernel Initialization**
4. **Initial RAM Disk (initrd/initramfs)**
5. **systemd Initialization**
6. **User Space Initialization**

## Stage 1: Power-On Self-Test (POST)

When a computer is powered on, the BIOS/UEFI performs a Power-On Self-Test (POST) to check hardware components:

- Tests CPU, memory, and basic hardware
- Initializes hardware components
- Determines the boot device based on the boot order in BIOS/UEFI settings

## Stage 2: Boot Loader (GRUB2)

### GRUB2 (GRand Unified Bootloader version 2)

GRUB2 is the default boot loader for most Linux distributions. It performs several functions:

- Displays a boot menu with available operating systems and kernel options
- Loads the selected kernel into memory
- Passes control to the kernel with optional parameters

### GRUB2 Configuration Files

```bash
# Main configuration file (do not edit directly)
/boot/grub2/grub.cfg

# Custom configuration
/etc/default/grub

# Custom menu entries
/etc/grub.d/
```

### Modifying GRUB2 Configuration

```bash
# Edit GRUB defaults
vi /etc/default/grub

# Common settings:
# GRUB_TIMEOUT=5
# GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
# GRUB_DEFAULT=saved
# GRUB_DISABLE_SUBMENU=true
# GRUB_TERMINAL_OUTPUT="console"
# GRUB_CMDLINE_LINUX="rhgb quiet"
# GRUB_DISABLE_RECOVERY="true"

# Regenerate grub.cfg after changes
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### GRUB2 Command Line

The GRUB command line can be accessed by pressing 'c' at the GRUB menu. Common commands include:

```
# List available devices
ls

# Set root device
set root=(hd0,1)

# Load Linux kernel
linux /vmlinuz-5.14.0-70.el9.x86_64 root=/dev/mapper/rhel-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M

# Load initial RAM disk
initrd /initramfs-5.14.0-70.el9.x86_64.img

# Boot the system
boot
```

## Stage 3: Kernel Initialization

After GRUB loads the kernel, the kernel performs several initialization tasks:

- Initializes memory management
- Detects and initializes CPU(s)
- Mounts the root filesystem in read-only mode initially
- Initializes essential hardware
- Starts the initial process (traditionally init, now systemd in most distributions)

### Kernel Parameters

Kernel parameters can be passed from GRUB to modify kernel behavior:

```
# Common kernel parameters:
# root=/dev/mapper/rhel-root - Specifies the root filesystem
# ro - Mounts root filesystem as read-only initially
# rhgb - Red Hat Graphical Boot (shows graphical boot screen)
# quiet - Suppresses most boot messages
# emergency - Boots into emergency mode
# single or 1 - Boots into single-user mode
# rd.break - Breaks before control is transferred to the init system
```

## Stage 4: Initial RAM Disk (initrd/initramfs)

The initial RAM disk (initrd) or initial RAM filesystem (initramfs) is a temporary root filesystem used during boot:

- Contains essential drivers and modules needed to mount the real root filesystem
- Includes tools for setting up devices, RAID, LVM, etc.
- Prepares the system for transitioning to the actual root filesystem

### Managing initramfs

```bash
# View contents of an initramfs
mkdir /tmp/initramfs
cd /tmp/initramfs
lsinitrd /boot/initramfs-$(uname -r).img

# Create or update an initramfs
dracut -f /boot/initramfs-$(uname -r).img $(uname -r)

# Create an initramfs with specific modules
dracut --add-drivers "xfs raid1" -f /boot/initramfs-$(uname -r).img $(uname -r)
```

## Stage 5: systemd Initialization

In modern Linux distributions, systemd is the first process (PID 1) started by the kernel:

- Reads its configuration from `/etc/systemd/system.conf`
- Determines the target to boot into (default is usually `multi-user.target` or `graphical.target`)
- Activates services and mounts filesystems based on dependencies

### systemd Targets

Targets are groups of units that represent a system state, similar to runlevels in SysV init:

```bash
# View default target
systemctl get-default

# Set default target
systemctl set-default multi-user.target

# List available targets
systemctl list-units --type=target
```

Common targets include:
- `emergency.target`: Minimal emergency shell
- `rescue.target`: Basic system initialization with rescue shell
- `multi-user.target`: Multi-user, non-graphical system
- `graphical.target`: Multi-user, graphical system

### systemd Boot Process

1. systemd reads `/etc/systemd/system.conf`
2. Activates all units required for the default target
3. Mounts all filesystems in `/etc/fstab`
4. Starts services based on dependencies and ordering

## Stage 6: User Space Initialization

After systemd initializes the system, various user space processes start:

- System services (networking, logging, etc.)
- Login manager (getty for console, display manager for GUI)
- User session initialization

## Boot Troubleshooting

### Modifying Boot Parameters at Runtime

1. At the GRUB menu, press 'e' to edit the selected entry
2. Find the line starting with `linux` or `linux16`
3. Add or modify parameters as needed
4. Press Ctrl+X to boot with the modified parameters

### Common Boot Issues and Solutions

#### Forgotten Root Password

```bash
# At GRUB menu, press 'e' to edit
# Add 'rd.break' to the kernel parameters
# Press Ctrl+X to boot
# At the emergency shell:
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
# Set new password
touch /.autorelabel
exit
exit
```

#### Filesystem Issues

```bash
# Boot into rescue mode by adding 'rescue' to kernel parameters
# Check and repair filesystems
fsck /dev/sda1
```

#### Failed Service Preventing Boot

```bash
# Add 'systemd.unit=emergency.target' to kernel parameters
# Disable the problematic service
systemctl disable problematic_service
reboot
```

### Using Rescue Mode

```bash
# Boot from installation media
# Select "Rescue a Red Hat system"
# Follow prompts to mount the system
# Chroot into the system
chroot /mnt/sysimage
```

## Boot Process Analysis

### Analyzing Boot Time

```bash
# View overall boot time
systemd-analyze

# View time spent in kernel
systemd-analyze time

# View time spent by each service
systemd-analyze blame

# Create a visualization of the boot process
systemd-analyze plot > boot.svg
```

### Viewing Boot Logs

```bash
# View kernel boot messages
dmesg

# View systemd boot logs
journalctl -b

# View logs from a previous boot
journalctl -b -1
```

## Boot Configuration Files

### Key Configuration Files

```bash
# GRUB configuration
/etc/default/grub
/boot/grub2/grub.cfg

# systemd configuration
/etc/systemd/system.conf
/etc/systemd/user.conf

# System initialization
/etc/fstab          # Filesystem mounts
/etc/crypttab       # Encrypted devices
/etc/vconsole.conf  # Virtual console settings
/etc/locale.conf    # System locale settings
/etc/hostname       # System hostname
```

## UEFI vs. BIOS Boot

### UEFI Boot Process

In UEFI systems:
- GRUB2 is installed to the EFI System Partition (ESP), typically mounted at `/boot/efi`
- The ESP contains the GRUB2 EFI application (grubx64.efi)
- Secure Boot may be enabled, requiring signed bootloaders and kernels

```bash
# View EFI variables
efibootmgr -v

# Create a new UEFI boot entry
efibootmgr --create --disk /dev/sda --part 1 --label "RHEL" --loader /EFI/redhat/grubx64.efi
```

### BIOS Boot Process

In BIOS systems:
- GRUB2 is installed to the Master Boot Record (MBR) or a dedicated BIOS Boot Partition
- The MBR contains the first stage of GRUB2, which loads the rest from `/boot`

## Dual-Boot Configuration

For systems with multiple operating systems:

```bash
# Detect other operating systems
os-prober

# Update GRUB configuration to include detected OS
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Best Practices

1. Keep multiple kernel versions installed for fallback
2. Test changes to boot configuration before rebooting
3. Create regular backups of boot configuration files
4. Document custom boot parameters
5. Understand rescue and emergency boot options
6. Keep bootable rescue media available
7. Monitor boot time performance regularly
8. Secure the GRUB bootloader with a password for sensitive systems
9. Maintain consistent filesystem labels and UUIDs
10. Test recovery procedures periodically