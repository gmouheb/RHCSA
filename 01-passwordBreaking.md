# Password Breaking in RHEL/CentOS

This document describes methods to reset the root password when you've lost access to a Red Hat Enterprise Linux or CentOS system.

## Method 1: Using rd.break

1. Interrupt the boot process by pressing 'e' at the GRUB menu
2. Add `rd.break` at the end of the line that starts with "linux" (before `rhgb quiet`)
3. Press Ctrl+X to boot with these parameters
4. Once in emergency mode, run the following commands:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
echo "password" | passwd --stdin root
touch /.autorelabel
exit
exit
```

The `/.autorelabel` file ensures SELinux contexts are properly restored on reboot.

## Method 2: Using init=/bin/bash

1. Interrupt the boot process by pressing 'e' at the GRUB menu
2. Add `init=/bin/bash` before `rhgb quiet`
3. Press Ctrl+X to boot with these parameters
4. Once in bash shell, run the following commands:

```bash
mount -o remount,rw /
passwd root
# Enter new password when prompted
touch /.autorelabel
exec /usr/lib/systemd/systemd
```

## Important Notes

- These methods should only be used for legitimate password recovery
- The `/.autorelabel` file is crucial if SELinux is enabled on your system
- After using either method, the system will reboot and relabel all files, which may take some time