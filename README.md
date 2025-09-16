# meilisearch Installation Guide

meilisearch is a free and open-source search API. Meilisearch provides instant search with typo tolerance

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 10GB for indices
  - Network: HTTP/REST API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 7700 (default meilisearch port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install meilisearch
sudo dnf install -y meilisearch

# Enable and start service
sudo systemctl enable --now meilisearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=7700/tcp
sudo firewall-cmd --reload

# Verify installation
meilisearch --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install meilisearch
sudo apt install -y meilisearch

# Enable and start service
sudo systemctl enable --now meilisearch

# Configure firewall
sudo ufw allow 7700

# Verify installation
meilisearch --version
```

### Arch Linux

```bash
# Install meilisearch
sudo pacman -S meilisearch

# Enable and start service
sudo systemctl enable --now meilisearch

# Verify installation
meilisearch --version
```

### Alpine Linux

```bash
# Install meilisearch
apk add --no-cache meilisearch

# Enable and start service
rc-update add meilisearch default
rc-service meilisearch start

# Verify installation
meilisearch --version
```

### openSUSE/SLES

```bash
# Install meilisearch
sudo zypper install -y meilisearch

# Enable and start service
sudo systemctl enable --now meilisearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=7700/tcp
sudo firewall-cmd --reload

# Verify installation
meilisearch --version
```

### macOS

```bash
# Using Homebrew
brew install meilisearch

# Start service
brew services start meilisearch

# Verify installation
meilisearch --version
```

### FreeBSD

```bash
# Using pkg
pkg install meilisearch

# Enable in rc.conf
echo 'meilisearch_enable="YES"' >> /etc/rc.conf

# Start service
service meilisearch start

# Verify installation
meilisearch --version
```

### Windows

```bash
# Using Chocolatey
choco install meilisearch

# Or using Scoop
scoop install meilisearch

# Verify installation
meilisearch --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/meilisearch

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
meilisearch --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable meilisearch

# Start service
sudo systemctl start meilisearch

# Stop service
sudo systemctl stop meilisearch

# Restart service
sudo systemctl restart meilisearch

# Check status
sudo systemctl status meilisearch

# View logs
sudo journalctl -u meilisearch -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add meilisearch default

# Start service
rc-service meilisearch start

# Stop service
rc-service meilisearch stop

# Restart service
rc-service meilisearch restart

# Check status
rc-service meilisearch status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'meilisearch_enable="YES"' >> /etc/rc.conf

# Start service
service meilisearch start

# Stop service
service meilisearch stop

# Restart service
service meilisearch restart

# Check status
service meilisearch status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start meilisearch
brew services stop meilisearch
brew services restart meilisearch

# Check status
brew services list | grep meilisearch
```

### Windows Service Manager

```powershell
# Start service
net start meilisearch

# Stop service
net stop meilisearch

# Using PowerShell
Start-Service meilisearch
Stop-Service meilisearch
Restart-Service meilisearch

# Check status
Get-Service meilisearch
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream meilisearch_backend {
    server 127.0.0.1:7700;
}

server {
    listen 80;
    server_name meilisearch.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name meilisearch.example.com;

    ssl_certificate /etc/ssl/certs/meilisearch.example.com.crt;
    ssl_certificate_key /etc/ssl/private/meilisearch.example.com.key;

    location / {
        proxy_pass http://meilisearch_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName meilisearch.example.com
    Redirect permanent / https://meilisearch.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName meilisearch.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/meilisearch.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/meilisearch.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:7700/
    ProxyPassReverse / http://127.0.0.1:7700/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend meilisearch_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/meilisearch.pem
    redirect scheme https if !{ ssl_fc }
    default_backend meilisearch_backend

backend meilisearch_backend
    balance roundrobin
    server meilisearch1 127.0.0.1:7700 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R meilisearch:meilisearch /etc/meilisearch
sudo chmod 750 /etc/meilisearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=7700/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status meilisearch

# View logs
sudo journalctl -u meilisearch -f

# Monitor resource usage
top -p $(pgrep meilisearch)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/meilisearch"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/meilisearch-backup-$DATE.tar.gz" /etc/meilisearch /var/lib/meilisearch

echo "Backup completed: $BACKUP_DIR/meilisearch-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop meilisearch

# Restore from backup
tar -xzf /backup/meilisearch/meilisearch-backup-*.tar.gz -C /

# Start service
sudo systemctl start meilisearch
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u meilisearch -n 100
sudo tail -f /var/log/meilisearch/meilisearch.log

# Check configuration
meilisearch --version

# Check permissions
ls -la /etc/meilisearch
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 7700

# Test connectivity
telnet localhost 7700

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep meilisearch)

# Check disk I/O
iotop -p $(pgrep meilisearch)

# Check connections
ss -an | grep 7700
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  meilisearch:
    image: meilisearch:latest
    ports:
      - "7700:7700"
    volumes:
      - ./config:/etc/meilisearch
      - ./data:/var/lib/meilisearch
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update meilisearch

# Debian/Ubuntu
sudo apt update && sudo apt upgrade meilisearch

# Arch Linux
sudo pacman -Syu meilisearch

# Alpine Linux
apk update && apk upgrade meilisearch

# openSUSE
sudo zypper update meilisearch

# FreeBSD
pkg update && pkg upgrade meilisearch

# Always backup before updates
tar -czf /backup/meilisearch-pre-update-$(date +%Y%m%d).tar.gz /etc/meilisearch

# Restart after updates
sudo systemctl restart meilisearch
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/meilisearch

# Clean old logs
find /var/log/meilisearch -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/meilisearch
```

## Additional Resources

- Official Documentation: https://docs.meilisearch.org/
- GitHub Repository: https://github.com/meilisearch/meilisearch
- Community Forum: https://forum.meilisearch.org/
- Best Practices Guide: https://docs.meilisearch.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
