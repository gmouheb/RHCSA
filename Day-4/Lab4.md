# Lab 4: Systemd Service Configuration

This lab focuses on installing and configuring systemd services, including dependency management and automatic restart settings.

## Tasks

1. Install the `vsftpd` and `httpd` services.

2. Set the default systemctl editor to vim.

3. Edit the httpd.service unit file such that starting httpd will always auto-start vsftpd.

4. Edit the httpd service such that after failure it will automatically restart in 10 seconds.