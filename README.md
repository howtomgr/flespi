# flespi Installation Guide

flespi is a free and open-source telematics gateway. Flespi provides telematics gateway platform

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
  - Storage: 20GB for data
  - Network: MQTT/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1883 (default flespi port)
  - HTTPS on 443
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

# Install flespi
sudo dnf install -y flespi

# Enable and start service
sudo systemctl enable --now flespi

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
flespi --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install flespi
sudo apt install -y flespi

# Enable and start service
sudo systemctl enable --now flespi

# Configure firewall
sudo ufw allow 1883

# Verify installation
flespi --version
```

### Arch Linux

```bash
# Install flespi
sudo pacman -S flespi

# Enable and start service
sudo systemctl enable --now flespi

# Verify installation
flespi --version
```

### Alpine Linux

```bash
# Install flespi
apk add --no-cache flespi

# Enable and start service
rc-update add flespi default
rc-service flespi start

# Verify installation
flespi --version
```

### openSUSE/SLES

```bash
# Install flespi
sudo zypper install -y flespi

# Enable and start service
sudo systemctl enable --now flespi

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
flespi --version
```

### macOS

```bash
# Using Homebrew
brew install flespi

# Start service
brew services start flespi

# Verify installation
flespi --version
```

### FreeBSD

```bash
# Using pkg
pkg install flespi

# Enable in rc.conf
echo 'flespi_enable="YES"' >> /etc/rc.conf

# Start service
service flespi start

# Verify installation
flespi --version
```

### Windows

```bash
# Using Chocolatey
choco install flespi

# Or using Scoop
scoop install flespi

# Verify installation
flespi --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/flespi

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
flespi --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable flespi

# Start service
sudo systemctl start flespi

# Stop service
sudo systemctl stop flespi

# Restart service
sudo systemctl restart flespi

# Check status
sudo systemctl status flespi

# View logs
sudo journalctl -u flespi -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add flespi default

# Start service
rc-service flespi start

# Stop service
rc-service flespi stop

# Restart service
rc-service flespi restart

# Check status
rc-service flespi status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'flespi_enable="YES"' >> /etc/rc.conf

# Start service
service flespi start

# Stop service
service flespi stop

# Restart service
service flespi restart

# Check status
service flespi status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start flespi
brew services stop flespi
brew services restart flespi

# Check status
brew services list | grep flespi
```

### Windows Service Manager

```powershell
# Start service
net start flespi

# Stop service
net stop flespi

# Using PowerShell
Start-Service flespi
Stop-Service flespi
Restart-Service flespi

# Check status
Get-Service flespi
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream flespi_backend {
    server 127.0.0.1:1883;
}

server {
    listen 80;
    server_name flespi.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name flespi.example.com;

    ssl_certificate /etc/ssl/certs/flespi.example.com.crt;
    ssl_certificate_key /etc/ssl/private/flespi.example.com.key;

    location / {
        proxy_pass http://flespi_backend;
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
    ServerName flespi.example.com
    Redirect permanent / https://flespi.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName flespi.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/flespi.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/flespi.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1883/
    ProxyPassReverse / http://127.0.0.1:1883/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend flespi_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/flespi.pem
    redirect scheme https if !{ ssl_fc }
    default_backend flespi_backend

backend flespi_backend
    balance roundrobin
    server flespi1 127.0.0.1:1883 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R flespi:flespi /etc/flespi
sudo chmod 750 /etc/flespi

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
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
sudo systemctl status flespi

# View logs
sudo journalctl -u flespi -f

# Monitor resource usage
top -p $(pgrep flespi)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/flespi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/flespi-backup-$DATE.tar.gz" /etc/flespi /var/lib/flespi

echo "Backup completed: $BACKUP_DIR/flespi-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop flespi

# Restore from backup
tar -xzf /backup/flespi/flespi-backup-*.tar.gz -C /

# Start service
sudo systemctl start flespi
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u flespi -n 100
sudo tail -f /var/log/flespi/flespi.log

# Check configuration
flespi --version

# Check permissions
ls -la /etc/flespi
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1883

# Test connectivity
telnet localhost 1883

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep flespi)

# Check disk I/O
iotop -p $(pgrep flespi)

# Check connections
ss -an | grep 1883
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  flespi:
    image: flespi:latest
    ports:
      - "1883:1883"
    volumes:
      - ./config:/etc/flespi
      - ./data:/var/lib/flespi
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update flespi

# Debian/Ubuntu
sudo apt update && sudo apt upgrade flespi

# Arch Linux
sudo pacman -Syu flespi

# Alpine Linux
apk update && apk upgrade flespi

# openSUSE
sudo zypper update flespi

# FreeBSD
pkg update && pkg upgrade flespi

# Always backup before updates
tar -czf /backup/flespi-pre-update-$(date +%Y%m%d).tar.gz /etc/flespi

# Restart after updates
sudo systemctl restart flespi
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/flespi

# Clean old logs
find /var/log/flespi -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/flespi
```

## Additional Resources

- Official Documentation: https://docs.flespi.org/
- GitHub Repository: https://github.com/flespi/flespi
- Community Forum: https://forum.flespi.org/
- Best Practices Guide: https://docs.flespi.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
