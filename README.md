# chirpstack Installation Guide

chirpstack is a free and open-source LoRaWAN server. ChirpStack provides open-source LoRaWAN Network Server

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: HTTP/LoRa
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default chirpstack port)
  - LoRa on 1700
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

# Install chirpstack
sudo dnf install -y chirpstack

# Enable and start service
sudo systemctl enable --now chirpstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
chirpstack --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install chirpstack
sudo apt install -y chirpstack

# Enable and start service
sudo systemctl enable --now chirpstack

# Configure firewall
sudo ufw allow 8080

# Verify installation
chirpstack --version
```

### Arch Linux

```bash
# Install chirpstack
sudo pacman -S chirpstack

# Enable and start service
sudo systemctl enable --now chirpstack

# Verify installation
chirpstack --version
```

### Alpine Linux

```bash
# Install chirpstack
apk add --no-cache chirpstack

# Enable and start service
rc-update add chirpstack default
rc-service chirpstack start

# Verify installation
chirpstack --version
```

### openSUSE/SLES

```bash
# Install chirpstack
sudo zypper install -y chirpstack

# Enable and start service
sudo systemctl enable --now chirpstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
chirpstack --version
```

### macOS

```bash
# Using Homebrew
brew install chirpstack

# Start service
brew services start chirpstack

# Verify installation
chirpstack --version
```

### FreeBSD

```bash
# Using pkg
pkg install chirpstack

# Enable in rc.conf
echo 'chirpstack_enable="YES"' >> /etc/rc.conf

# Start service
service chirpstack start

# Verify installation
chirpstack --version
```

### Windows

```bash
# Using Chocolatey
choco install chirpstack

# Or using Scoop
scoop install chirpstack

# Verify installation
chirpstack --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/chirpstack

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
chirpstack --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable chirpstack

# Start service
sudo systemctl start chirpstack

# Stop service
sudo systemctl stop chirpstack

# Restart service
sudo systemctl restart chirpstack

# Check status
sudo systemctl status chirpstack

# View logs
sudo journalctl -u chirpstack -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add chirpstack default

# Start service
rc-service chirpstack start

# Stop service
rc-service chirpstack stop

# Restart service
rc-service chirpstack restart

# Check status
rc-service chirpstack status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'chirpstack_enable="YES"' >> /etc/rc.conf

# Start service
service chirpstack start

# Stop service
service chirpstack stop

# Restart service
service chirpstack restart

# Check status
service chirpstack status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start chirpstack
brew services stop chirpstack
brew services restart chirpstack

# Check status
brew services list | grep chirpstack
```

### Windows Service Manager

```powershell
# Start service
net start chirpstack

# Stop service
net stop chirpstack

# Using PowerShell
Start-Service chirpstack
Stop-Service chirpstack
Restart-Service chirpstack

# Check status
Get-Service chirpstack
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream chirpstack_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name chirpstack.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name chirpstack.example.com;

    ssl_certificate /etc/ssl/certs/chirpstack.example.com.crt;
    ssl_certificate_key /etc/ssl/private/chirpstack.example.com.key;

    location / {
        proxy_pass http://chirpstack_backend;
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
    ServerName chirpstack.example.com
    Redirect permanent / https://chirpstack.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName chirpstack.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/chirpstack.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/chirpstack.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend chirpstack_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/chirpstack.pem
    redirect scheme https if !{ ssl_fc }
    default_backend chirpstack_backend

backend chirpstack_backend
    balance roundrobin
    server chirpstack1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R chirpstack:chirpstack /etc/chirpstack
sudo chmod 750 /etc/chirpstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status chirpstack

# View logs
sudo journalctl -u chirpstack -f

# Monitor resource usage
top -p $(pgrep chirpstack)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/chirpstack"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/chirpstack-backup-$DATE.tar.gz" /etc/chirpstack /var/lib/chirpstack

echo "Backup completed: $BACKUP_DIR/chirpstack-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop chirpstack

# Restore from backup
tar -xzf /backup/chirpstack/chirpstack-backup-*.tar.gz -C /

# Start service
sudo systemctl start chirpstack
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u chirpstack -n 100
sudo tail -f /var/log/chirpstack/chirpstack.log

# Check configuration
chirpstack --version

# Check permissions
ls -la /etc/chirpstack
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep chirpstack)

# Check disk I/O
iotop -p $(pgrep chirpstack)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  chirpstack:
    image: chirpstack:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/chirpstack
      - ./data:/var/lib/chirpstack
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update chirpstack

# Debian/Ubuntu
sudo apt update && sudo apt upgrade chirpstack

# Arch Linux
sudo pacman -Syu chirpstack

# Alpine Linux
apk update && apk upgrade chirpstack

# openSUSE
sudo zypper update chirpstack

# FreeBSD
pkg update && pkg upgrade chirpstack

# Always backup before updates
tar -czf /backup/chirpstack-pre-update-$(date +%Y%m%d).tar.gz /etc/chirpstack

# Restart after updates
sudo systemctl restart chirpstack
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/chirpstack

# Clean old logs
find /var/log/chirpstack -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/chirpstack
```

## Additional Resources

- Official Documentation: https://docs.chirpstack.org/
- GitHub Repository: https://github.com/chirpstack/chirpstack
- Community Forum: https://forum.chirpstack.org/
- Best Practices Guide: https://docs.chirpstack.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
