# proxmox-ve Installation Guide

proxmox-ve is a free and open-source virtualization platform. Proxmox VE combines KVM and LXC for enterprise virtualization

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
  - CPU: 4+ cores
  - RAM: 8GB minimum
  - Storage: 32GB for VMs
  - Network: Cluster networking
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8006 (default proxmox-ve port)
  - Various cluster ports
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

# Install proxmox-ve
sudo dnf install -y proxmox-ve

# Enable and start service
sudo systemctl enable --now proxmox-ve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8006/tcp
sudo firewall-cmd --reload

# Verify installation
proxmox-ve --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install proxmox-ve
sudo apt install -y proxmox-ve

# Enable and start service
sudo systemctl enable --now proxmox-ve

# Configure firewall
sudo ufw allow 8006

# Verify installation
proxmox-ve --version
```

### Arch Linux

```bash
# Install proxmox-ve
sudo pacman -S proxmox-ve

# Enable and start service
sudo systemctl enable --now proxmox-ve

# Verify installation
proxmox-ve --version
```

### Alpine Linux

```bash
# Install proxmox-ve
apk add --no-cache proxmox-ve

# Enable and start service
rc-update add proxmox-ve default
rc-service proxmox-ve start

# Verify installation
proxmox-ve --version
```

### openSUSE/SLES

```bash
# Install proxmox-ve
sudo zypper install -y proxmox-ve

# Enable and start service
sudo systemctl enable --now proxmox-ve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8006/tcp
sudo firewall-cmd --reload

# Verify installation
proxmox-ve --version
```

### macOS

```bash
# Using Homebrew
brew install proxmox-ve

# Start service
brew services start proxmox-ve

# Verify installation
proxmox-ve --version
```

### FreeBSD

```bash
# Using pkg
pkg install proxmox-ve

# Enable in rc.conf
echo 'proxmox-ve_enable="YES"' >> /etc/rc.conf

# Start service
service proxmox-ve start

# Verify installation
proxmox-ve --version
```

### Windows

```bash
# Using Chocolatey
choco install proxmox-ve

# Or using Scoop
scoop install proxmox-ve

# Verify installation
proxmox-ve --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/proxmox-ve

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
proxmox-ve --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable proxmox-ve

# Start service
sudo systemctl start proxmox-ve

# Stop service
sudo systemctl stop proxmox-ve

# Restart service
sudo systemctl restart proxmox-ve

# Check status
sudo systemctl status proxmox-ve

# View logs
sudo journalctl -u proxmox-ve -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add proxmox-ve default

# Start service
rc-service proxmox-ve start

# Stop service
rc-service proxmox-ve stop

# Restart service
rc-service proxmox-ve restart

# Check status
rc-service proxmox-ve status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'proxmox-ve_enable="YES"' >> /etc/rc.conf

# Start service
service proxmox-ve start

# Stop service
service proxmox-ve stop

# Restart service
service proxmox-ve restart

# Check status
service proxmox-ve status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start proxmox-ve
brew services stop proxmox-ve
brew services restart proxmox-ve

# Check status
brew services list | grep proxmox-ve
```

### Windows Service Manager

```powershell
# Start service
net start proxmox-ve

# Stop service
net stop proxmox-ve

# Using PowerShell
Start-Service proxmox-ve
Stop-Service proxmox-ve
Restart-Service proxmox-ve

# Check status
Get-Service proxmox-ve
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream proxmox-ve_backend {
    server 127.0.0.1:8006;
}

server {
    listen 80;
    server_name proxmox-ve.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name proxmox-ve.example.com;

    ssl_certificate /etc/ssl/certs/proxmox-ve.example.com.crt;
    ssl_certificate_key /etc/ssl/private/proxmox-ve.example.com.key;

    location / {
        proxy_pass http://proxmox-ve_backend;
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
    ServerName proxmox-ve.example.com
    Redirect permanent / https://proxmox-ve.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName proxmox-ve.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/proxmox-ve.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/proxmox-ve.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8006/
    ProxyPassReverse / http://127.0.0.1:8006/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend proxmox-ve_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/proxmox-ve.pem
    redirect scheme https if !{ ssl_fc }
    default_backend proxmox-ve_backend

backend proxmox-ve_backend
    balance roundrobin
    server proxmox-ve1 127.0.0.1:8006 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R proxmox-ve:proxmox-ve /etc/proxmox-ve
sudo chmod 750 /etc/proxmox-ve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8006/tcp
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
sudo systemctl status proxmox-ve

# View logs
sudo journalctl -u proxmox-ve -f

# Monitor resource usage
top -p $(pgrep proxmox-ve)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/proxmox-ve"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/proxmox-ve-backup-$DATE.tar.gz" /etc/proxmox-ve /var/lib/proxmox-ve

echo "Backup completed: $BACKUP_DIR/proxmox-ve-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop proxmox-ve

# Restore from backup
tar -xzf /backup/proxmox-ve/proxmox-ve-backup-*.tar.gz -C /

# Start service
sudo systemctl start proxmox-ve
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u proxmox-ve -n 100
sudo tail -f /var/log/proxmox-ve/proxmox-ve.log

# Check configuration
proxmox-ve --version

# Check permissions
ls -la /etc/proxmox-ve
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8006

# Test connectivity
telnet localhost 8006

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep proxmox-ve)

# Check disk I/O
iotop -p $(pgrep proxmox-ve)

# Check connections
ss -an | grep 8006
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  proxmox-ve:
    image: proxmox-ve:latest
    ports:
      - "8006:8006"
    volumes:
      - ./config:/etc/proxmox-ve
      - ./data:/var/lib/proxmox-ve
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update proxmox-ve

# Debian/Ubuntu
sudo apt update && sudo apt upgrade proxmox-ve

# Arch Linux
sudo pacman -Syu proxmox-ve

# Alpine Linux
apk update && apk upgrade proxmox-ve

# openSUSE
sudo zypper update proxmox-ve

# FreeBSD
pkg update && pkg upgrade proxmox-ve

# Always backup before updates
tar -czf /backup/proxmox-ve-pre-update-$(date +%Y%m%d).tar.gz /etc/proxmox-ve

# Restart after updates
sudo systemctl restart proxmox-ve
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/proxmox-ve

# Clean old logs
find /var/log/proxmox-ve -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/proxmox-ve
```

## Additional Resources

- Official Documentation: https://docs.proxmox-ve.org/
- GitHub Repository: https://github.com/proxmox-ve/proxmox-ve
- Community Forum: https://forum.proxmox-ve.org/
- Best Practices Guide: https://docs.proxmox-ve.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
