# suricata Installation Guide

suricata is a free and open-source high performance Network IDS/IPS. Suricata provides network security monitoring with IDS/IPS capabilities, serving as an open-source alternative to commercial solutions like Cisco IPS

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (8GB+ for high traffic)
  - Storage: 10GB for logs
  - Network: Network tap or mirror port
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default suricata port)
  - Unix socket for EVE
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

# Install suricata
sudo dnf install -y suricata

# Enable and start service
sudo systemctl enable --now suricata

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
suricata --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install suricata
sudo apt install -y suricata

# Enable and start service
sudo systemctl enable --now suricata

# Configure firewall
sudo ufw allow N/A

# Verify installation
suricata --version
```

### Arch Linux

```bash
# Install suricata
sudo pacman -S suricata

# Enable and start service
sudo systemctl enable --now suricata

# Verify installation
suricata --version
```

### Alpine Linux

```bash
# Install suricata
apk add --no-cache suricata

# Enable and start service
rc-update add suricata default
rc-service suricata start

# Verify installation
suricata --version
```

### openSUSE/SLES

```bash
# Install suricata
sudo zypper install -y suricata

# Enable and start service
sudo systemctl enable --now suricata

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
suricata --version
```

### macOS

```bash
# Using Homebrew
brew install suricata

# Start service
brew services start suricata

# Verify installation
suricata --version
```

### FreeBSD

```bash
# Using pkg
pkg install suricata

# Enable in rc.conf
echo 'suricata_enable="YES"' >> /etc/rc.conf

# Start service
service suricata start

# Verify installation
suricata --version
```

### Windows

```bash
# Using Chocolatey
choco install suricata

# Or using Scoop
scoop install suricata

# Verify installation
suricata --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/suricata

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
suricata --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable suricata

# Start service
sudo systemctl start suricata

# Stop service
sudo systemctl stop suricata

# Restart service
sudo systemctl restart suricata

# Check status
sudo systemctl status suricata

# View logs
sudo journalctl -u suricata -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add suricata default

# Start service
rc-service suricata start

# Stop service
rc-service suricata stop

# Restart service
rc-service suricata restart

# Check status
rc-service suricata status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'suricata_enable="YES"' >> /etc/rc.conf

# Start service
service suricata start

# Stop service
service suricata stop

# Restart service
service suricata restart

# Check status
service suricata status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start suricata
brew services stop suricata
brew services restart suricata

# Check status
brew services list | grep suricata
```

### Windows Service Manager

```powershell
# Start service
net start suricata

# Stop service
net stop suricata

# Using PowerShell
Start-Service suricata
Stop-Service suricata
Restart-Service suricata

# Check status
Get-Service suricata
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream suricata_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name suricata.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name suricata.example.com;

    ssl_certificate /etc/ssl/certs/suricata.example.com.crt;
    ssl_certificate_key /etc/ssl/private/suricata.example.com.key;

    location / {
        proxy_pass http://suricata_backend;
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
    ServerName suricata.example.com
    Redirect permanent / https://suricata.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName suricata.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/suricata.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/suricata.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend suricata_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/suricata.pem
    redirect scheme https if !{ ssl_fc }
    default_backend suricata_backend

backend suricata_backend
    balance roundrobin
    server suricata1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R suricata:suricata /etc/suricata
sudo chmod 750 /etc/suricata

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
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
sudo systemctl status suricata

# View logs
sudo journalctl -u suricata -f

# Monitor resource usage
top -p $(pgrep suricata)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/suricata"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/suricata-backup-$DATE.tar.gz" /etc/suricata /var/lib/suricata

echo "Backup completed: $BACKUP_DIR/suricata-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop suricata

# Restore from backup
tar -xzf /backup/suricata/suricata-backup-*.tar.gz -C /

# Start service
sudo systemctl start suricata
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u suricata -n 100
sudo tail -f /var/log/suricata/suricata.log

# Check configuration
suricata --version

# Check permissions
ls -la /etc/suricata
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep N/A

# Test connectivity
telnet localhost N/A

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep suricata)

# Check disk I/O
iotop -p $(pgrep suricata)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  suricata:
    image: suricata:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/suricata
      - ./data:/var/lib/suricata
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update suricata

# Debian/Ubuntu
sudo apt update && sudo apt upgrade suricata

# Arch Linux
sudo pacman -Syu suricata

# Alpine Linux
apk update && apk upgrade suricata

# openSUSE
sudo zypper update suricata

# FreeBSD
pkg update && pkg upgrade suricata

# Always backup before updates
tar -czf /backup/suricata-pre-update-$(date +%Y%m%d).tar.gz /etc/suricata

# Restart after updates
sudo systemctl restart suricata
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/suricata

# Clean old logs
find /var/log/suricata -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/suricata
```

## Additional Resources

- Official Documentation: https://docs.suricata.org/
- GitHub Repository: https://github.com/suricata/suricata
- Community Forum: https://forum.suricata.org/
- Best Practices Guide: https://docs.suricata.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
