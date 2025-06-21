# HTTP Service in Linux

This document covers the installation, configuration, and management of HTTP servers in Linux, with a focus on Apache HTTP Server (httpd) in RHEL/CentOS environments.

## Introduction to HTTP Servers

HTTP (Hypertext Transfer Protocol) servers are the foundation of the World Wide Web, serving web content to clients. Common HTTP servers in Linux include:

- Apache HTTP Server (httpd)
- Nginx
- Lighttpd
- Caddy

This guide primarily focuses on Apache HTTP Server, which is the default web server in RHEL/CentOS systems.

## Installing Apache HTTP Server

### Installation on RHEL/CentOS

```bash
# Install Apache HTTP Server
dnf install httpd

# Start and enable the service
systemctl enable --now httpd

# Check service status
systemctl status httpd
```

### Firewall Configuration

```bash
# Allow HTTP traffic (port 80)
firewall-cmd --permanent --add-service=http

# Allow HTTPS traffic (port 443)
firewall-cmd --permanent --add-service=https

# Reload firewall
firewall-cmd --reload
```

## Apache Configuration Basics

### Configuration Files

Apache's configuration files are located in the following directories:

- Main configuration file: `/etc/httpd/conf/httpd.conf`
- Configuration directories:
  - `/etc/httpd/conf.d/` - Additional configuration files
  - `/etc/httpd/conf.modules.d/` - Module configuration files

### Basic Configuration Directives

```apache
# Server information
ServerRoot "/etc/httpd"
ServerName www.example.com:80
ServerAdmin webmaster@example.com

# Listening ports
Listen 80

# Document root
DocumentRoot "/var/www/html"

# Directory permissions
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

# Log files
ErrorLog "logs/error_log"
LogLevel warn
CustomLog "logs/access_log" combined
```

### Testing Configuration

```bash
# Test configuration syntax
apachectl configtest
# or
httpd -t

# Reload configuration
systemctl reload httpd

# Restart service
systemctl restart httpd
```

## Virtual Hosts

Virtual hosts allow a single Apache server to host multiple websites with different domain names.

### Name-based Virtual Hosts

```apache
# /etc/httpd/conf.d/vhost.conf

<VirtualHost *:80>
    ServerName www.example.com
    ServerAlias example.com
    DocumentRoot "/var/www/example.com/html"
    ErrorLog "logs/example.com-error_log"
    CustomLog "logs/example.com-access_log" combined
    
    <Directory "/var/www/example.com/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example.org
    ServerAlias example.org
    DocumentRoot "/var/www/example.org/html"
    ErrorLog "logs/example.org-error_log"
    CustomLog "logs/example.org-access_log" combined
    
    <Directory "/var/www/example.org/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

### IP-based Virtual Hosts

```apache
<VirtualHost 192.168.1.10:80>
    ServerName www.example.com
    DocumentRoot "/var/www/example.com/html"
</VirtualHost>

<VirtualHost 192.168.1.11:80>
    ServerName www.example.org
    DocumentRoot "/var/www/example.org/html"
</VirtualHost>
```

### Creating Directory Structure

```bash
# Create directories for virtual hosts
mkdir -p /var/www/example.com/html
mkdir -p /var/www/example.org/html

# Set ownership
chown -R apache:apache /var/www/example.com
chown -R apache:apache /var/www/example.org

# Set permissions
chmod -R 755 /var/www
```

## SSL/TLS Configuration

### Installing SSL Module

```bash
# Install mod_ssl
dnf install mod_ssl
```

### Creating Self-signed Certificate

```bash
# Generate private key and certificate
mkdir -p /etc/httpd/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/httpd/ssl/apache.key \
    -out /etc/httpd/ssl/apache.crt
```

### Configuring SSL Virtual Host

```apache
# /etc/httpd/conf.d/ssl.conf

<VirtualHost *:443>
    ServerName www.example.com
    DocumentRoot "/var/www/example.com/html"
    
    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/apache.crt
    SSLCertificateKeyFile /etc/httpd/ssl/apache.key
    
    <Directory "/var/www/example.com/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    ErrorLog "logs/ssl-example.com-error_log"
    CustomLog "logs/ssl-example.com-access_log" combined
</VirtualHost>
```

### HTTP to HTTPS Redirection

```apache
<VirtualHost *:80>
    ServerName www.example.com
    Redirect permanent / https://www.example.com/
</VirtualHost>
```

## Apache Modules

Apache's functionality can be extended with modules.

### Common Modules

- `mod_ssl` - SSL/TLS support
- `mod_rewrite` - URL rewriting
- `mod_proxy` - Proxy/gateway support
- `mod_security` - Web application firewall
- `mod_php` - PHP support
- `mod_wsgi` - Python WSGI support

### Managing Modules

```bash
# List loaded modules
httpd -M

# Enable a module (by creating a symlink)
ln -s /etc/httpd/modules/mod_example.so /etc/httpd/modules.d/00-example.conf

# Disable a module (by removing or commenting out the configuration)
```

### Example: Enabling mod_rewrite

```apache
# Ensure mod_rewrite is loaded
LoadModule rewrite_module modules/mod_rewrite.so

# Enable rewriting in a directory
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

## URL Rewriting with mod_rewrite

### Basic Rewrite Rules

```apache
# Enable rewrite engine
RewriteEngine On

# Redirect example.com to www.example.com
RewriteCond %{HTTP_HOST} ^example\.com$ [NC]
RewriteRule ^(.*)$ http://www.example.com/$1 [R=301,L]

# Rewrite URLs to hide .php extension
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}\.php -f
RewriteRule ^(.*)$ $1.php [L]
```

### Using .htaccess Files

```apache
# Allow .htaccess files in the configuration
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

Example `.htaccess` file:
```apache
RewriteEngine On
RewriteBase /

# Redirect non-www to www
RewriteCond %{HTTP_HOST} !^www\. [NC]
RewriteRule ^(.*)$ http://www.%{HTTP_HOST}/$1 [R=301,L]

# Rewrite for clean URLs
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([^/]+)/?$ index.php?page=$1 [L,QSA]
```

## Access Control

### IP-based Access Control

```apache
<Directory "/var/www/html/admin">
    # Allow from specific IP
    Require ip 192.168.1.100
    
    # Allow from IP range
    Require ip 10.0.0.0/24
    
    # Allow from multiple sources
    Require ip 192.168.1.100 10.0.0.0/24
</Directory>
```

### Password Authentication

```bash
# Create password file
htpasswd -c /etc/httpd/passwords username
```

```apache
<Directory "/var/www/html/protected">
    AuthType Basic
    AuthName "Restricted Area"
    AuthUserFile /etc/httpd/passwords
    Require valid-user
</Directory>
```

## Performance Tuning

### MPM (Multi-Processing Module) Configuration

Apache supports different MPMs for handling requests:

- `prefork` - Traditional non-threaded model
- `worker` - Hybrid multi-process and multi-threaded model
- `event` - Improved version of worker MPM

Example configuration for the event MPM:
```apache
# /etc/httpd/conf.modules.d/00-mpm.conf

LoadModule mpm_event_module modules/mod_mpm_event.so

<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>
```

### Caching Configuration

```apache
# Enable caching modules
LoadModule cache_module modules/mod_cache.so
LoadModule cache_disk_module modules/mod_cache_disk.so

# Configure disk cache
<IfModule mod_cache_disk.c>
    CacheRoot "/var/cache/httpd/proxy"
    CacheEnable disk /
    CacheDirLevels 2
    CacheDirLength 1
    CacheMaxFileSize 1000000
</IfModule>

# Set cache expiration
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/gif "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType text/css "access plus 1 week"
    ExpiresByType application/javascript "access plus 1 week"
    ExpiresDefault "access plus 2 days"
</IfModule>
```

## Logging and Monitoring

### Log Formats

```apache
# Define log formats
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common

# Apply log format
CustomLog "logs/access_log" combined
```

### Log Rotation

Apache logs are typically rotated using `logrotate`. The configuration is usually in `/etc/logrotate.d/httpd`:

```
/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```

### Monitoring Tools

- `apachetop` - Real-time monitoring of Apache logs
- `mod_status` - Apache status module
- `GoAccess` - Visual web log analyzer

Example `mod_status` configuration:
```apache
<Location "/server-status">
    SetHandler server-status
    Require ip 127.0.0.1
</Location>
```

## Proxying and Load Balancing

### Reverse Proxy Configuration

```apache
# Load required modules
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

# Basic reverse proxy
ProxyPass "/app/" "http://backend-server:8080/"
ProxyPassReverse "/app/" "http://backend-server:8080/"

# Proxy with additional options
<Location "/api">
    ProxyPass "http://api-server:8080/"
    ProxyPassReverse "http://api-server:8080/"
    ProxyPreserveHost On
</Location>
```

### Load Balancing

```apache
# Load balancer configuration
<Proxy "balancer://mycluster">
    BalancerMember "http://app1.example.com:8080"
    BalancerMember "http://app2.example.com:8080"
    ProxySet lbmethod=byrequests
</Proxy>

# Use the load balancer
ProxyPass "/app" "balancer://mycluster"
ProxyPassReverse "/app" "balancer://mycluster"
```

## Security Considerations

### Basic Security Measures

```apache
# Hide Apache version information
ServerTokens Prod
ServerSignature Off

# Disable directory listing
Options -Indexes

# Disable server-side includes and CGI execution
Options -Includes -ExecCGI

# Restrict access to .htaccess files
<Files ".ht*">
    Require all denied
</Files>

# Set secure headers
Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-XSS-Protection "1; mode=block"
```

### ModSecurity Web Application Firewall

```bash
# Install ModSecurity
dnf install mod_security mod_security_crs
```

Basic ModSecurity configuration:
```apache
# Load ModSecurity module
LoadModule security2_module modules/mod_security2.so

# Enable ModSecurity
<IfModule mod_security2.c>
    SecRuleEngine On
    SecRequestBodyAccess On
    SecResponseBodyAccess On
    SecResponseBodyMimeType text/plain text/html text/xml application/json
    SecDataDir /var/lib/mod_security
    
    # Include OWASP Core Rule Set
    Include modsecurity.d/activated_rules/*.conf
</IfModule>
```

## Troubleshooting

### Common Issues and Solutions

1. **Apache won't start**:
   - Check syntax errors: `httpd -t`
   - Check logs: `journalctl -u httpd`
   - Check for port conflicts: `ss -tulpn | grep :80`

2. **403 Forbidden errors**:
   - Check file permissions
   - Check SELinux context: `ls -Z /var/www/html`
   - Set correct SELinux context: `chcon -R -t httpd_sys_content_t /var/www/html`

3. **SSL certificate issues**:
   - Verify certificate paths
   - Check certificate validity: `openssl x509 -in certificate.crt -text -noout`

### Debugging Techniques

```bash
# Increase log level temporarily
LogLevel debug

# Test a specific virtual host
curl -H "Host: www.example.com" http://localhost/

# Check which configuration file is being used
httpd -V

# Check loaded modules
httpd -M
```

## Apache with PHP

### Installing PHP for Apache

```bash
# Install PHP and Apache PHP module
dnf install php php-mysqlnd php-gd php-xml php-mbstring

# Restart Apache
systemctl restart httpd
```

### PHP Configuration

```apache
# /etc/httpd/conf.d/php.conf

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

# PHP settings
php_value memory_limit 128M
php_value upload_max_filesize 20M
php_value post_max_size 20M
php_value max_execution_time 30
```

### Testing PHP

Create a test file `/var/www/html/info.php`:
```php
<?php
phpinfo();
?>
```

Access it at `http://your-server/info.php`

## Best Practices

1. **Keep Apache updated** to patch security vulnerabilities
2. **Use HTTPS** for all websites
3. **Implement proper access controls**
4. **Regularly review logs** for suspicious activity
5. **Use ModSecurity** or other WAF for additional protection
6. **Optimize performance** based on your workload
7. **Implement proper caching** strategies
8. **Use separate virtual hosts** for different websites
9. **Regularly backup configuration files**
10. **Document your configuration** changes