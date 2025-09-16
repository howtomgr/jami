# Jami Installation Guide

Jami is a free and open-source VoIP/Video. Free and universal communication platform

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5060/5061 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5060/5061 (default jami port)
- **Dependencies**:
  - libdbus-1-3, libavcodec58
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

# Install jami
sudo dnf install -y jami libdbus-1-3, libavcodec58

# Enable and start service
sudo systemctl enable --now jami

# Configure firewall
sudo firewall-cmd --permanent --add-service=jami
sudo firewall-cmd --reload

# Verify installation
jami --version || systemctl status jami
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install jami
sudo apt install -y jami libdbus-1-3, libavcodec58

# Enable and start service
sudo systemctl enable --now jami

# Configure firewall
sudo ufw allow 5060/5061

# Verify installation
jami --version || systemctl status jami
```

### Arch Linux

```bash
# Install jami
sudo pacman -S jami

# Enable and start service
sudo systemctl enable --now jami

# Verify installation
jami --version || systemctl status jami
```

### Alpine Linux

```bash
# Install jami
apk add --no-cache jami

# Enable and start service
rc-update add jami default
rc-service jami start

# Verify installation
jami --version || rc-service jami status
```

### openSUSE/SLES

```bash
# Install jami
sudo zypper install -y jami libdbus-1-3, libavcodec58

# Enable and start service
sudo systemctl enable --now jami

# Configure firewall
sudo firewall-cmd --permanent --add-service=jami
sudo firewall-cmd --reload

# Verify installation
jami --version || systemctl status jami
```

### macOS

```bash
# Using Homebrew
brew install jami

# Start service
brew services start jami

# Verify installation
jami --version
```

### FreeBSD

```bash
# Using pkg
pkg install jami

# Enable in rc.conf
echo 'jami_enable="YES"' >> /etc/rc.conf

# Start service
service jami start

# Verify installation
jami --version || service jami status
```

### Windows

```powershell
# Using Chocolatey
choco install jami

# Or using Scoop
scoop install jami

# Verify installation
jami --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/jami

# Set up basic configuration
sudo tee /etc/jami/jami.conf << 'EOF'
# Jami Configuration
ice_tcp_enable=true
EOF

# Test configuration
sudo jami -t || sudo jami configtest

# Reload service
sudo systemctl reload jami
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R jami:jami /etc/jami
sudo chmod 750 /etc/jami

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable jami

# Start service
sudo systemctl start jami

# Stop service
sudo systemctl stop jami

# Restart service
sudo systemctl restart jami

# Reload configuration
sudo systemctl reload jami

# Check status
sudo systemctl status jami

# View logs
sudo journalctl -u jami -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add jami default

# Start service
rc-service jami start

# Stop service
rc-service jami stop

# Restart service
rc-service jami restart

# Check status
rc-service jami status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'jami_enable="YES"' >> /etc/rc.conf

# Start service
service jami start

# Stop service
service jami stop

# Restart service
service jami restart

# Check status
service jami status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start jami
brew services stop jami
brew services restart jami

# Check status
brew services list | grep jami
```

### Windows Service Manager

```powershell
# Start service
net start jami

# Stop service
net stop jami

# Using PowerShell
Start-Service jami
Stop-Service jami
Restart-Service jami

# Check status
Get-Service jami
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/jami/jami.conf << 'EOF'
ice_tcp_enable=true
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart jami
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream jami_backend {
    server 127.0.0.1:5060/5061;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name jami.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name jami.example.com;

    ssl_certificate /etc/ssl/certs/jami.example.com.crt;
    ssl_certificate_key /etc/ssl/private/jami.example.com.key;

    location / {
        proxy_pass http://jami_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName jami.example.com
    Redirect permanent / https://jami.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName jami.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/jami.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/jami.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5060/5061/
    ProxyPassReverse / http://127.0.0.1:5060/5061/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5060/5061/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend jami_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/jami.pem
    redirect scheme https if !{ ssl_fc }
    default_backend jami_backend

backend jami_backend
    balance roundrobin
    option httpchk GET /health
    server jami1 127.0.0.1:5060/5061 check
    server jami2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R jami:jami /etc/jami
sudo chmod 750 /etc/jami

# Configure firewall
sudo firewall-cmd --permanent --add-service=jami
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/jami.conf << 'EOF'
[jami]
enabled = true
port = 5060/5061
filter = jami
logpath = /var/log/jami/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/jami.key \
    -out /etc/ssl/certs/jami.crt

# Configure SSL in jami
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE jami_db;
CREATE USER jami_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE jami_db TO jami_user;
EOF

# Configure jami to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE jami_db;
CREATE USER 'jami_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON jami_db.* TO 'jami_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Jami specific tuning
ice_tcp_enable=true
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
jami soft nofile 65535
jami hard nofile 65535
jami soft nproc 32768
jami hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'jami'
    static_configs:
      - targets: ['localhost:5060/5061']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet jami; then
    echo "Jami is running"
    exit 0
else
    echo "Jami is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/jami << 'EOF'
/var/log/jami/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 jami jami
    postrotate
        systemctl reload jami > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Jami backup script
BACKUP_DIR="/backup/jami"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop jami

# Backup configuration
tar -czf "$BACKUP_DIR/jami-config-$DATE.tar.gz" /etc/jami

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/jami-data-$DATE.tar.gz" /var/lib/jami

# Start service
systemctl start jami

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop jami

# Restore configuration
sudo tar -xzf /backup/jami/jami-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/jami/jami-data-*.tar.gz -C /

# Set permissions
sudo chown -R jami:jami /etc/jami
sudo chown -R jami:jami /var/lib/jami

# Start service
sudo systemctl start jami
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u jami -n 100
sudo tail -f /var/log/jami/*.log

# Check configuration
sudo jami -t || sudo jami configtest

# Check permissions
ls -la /etc/jami
ls -la /var/lib/jami
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5060/5061
sudo netstat -tlnp | grep 5060/5061

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5060/5061
nc -zv localhost 5060/5061
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep jamid)
htop -p $(pgrep jamid)

# Check connections
ss -ant | grep :5060/5061 | wc -l

# Monitor I/O
iotop -p $(pgrep jamid)
```

### Debug Mode

```bash
# Run in debug mode
sudo jami -d
# or
sudo jami debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  jami:
    image: jami:latest
    container_name: jami
    ports:
      - "5060/5061:5060/5061"
    volumes:
      - ./config:/etc/jami
      - ./data:/var/lib/jami
    environment:
      - jami_CONFIG=/etc/jami/jami.conf
    restart: unless-stopped
    networks:
      - jami_net

networks:
  jami_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jami
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jami
  template:
    metadata:
      labels:
        app: jami
    spec:
      containers:
      - name: jami
        image: jami:latest
        ports:
        - containerPort: 5060/5061
        volumeMounts:
        - name: config
          mountPath: /etc/jami
      volumes:
      - name: config
        configMap:
          name: jami-config
---
apiVersion: v1
kind: Service
metadata:
  name: jami
spec:
  selector:
    app: jami
  ports:
  - port: 5060/5061
    targetPort: 5060/5061
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Jami
  hosts: all
  become: yes
  tasks:
    - name: Install jami
      package:
        name: jami
        state: present
    
    - name: Configure jami
      template:
        src: jami.conf.j2
        dest: /etc/jami/jami.conf
        owner: jami
        group: jami
        mode: '0640'
      notify: restart jami
    
    - name: Start and enable jami
      systemd:
        name: jami
        state: started
        enabled: yes
  
  handlers:
    - name: restart jami
      systemd:
        name: jami
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update jami

# Debian/Ubuntu
sudo apt update && sudo apt upgrade jami

# Arch Linux
sudo pacman -Syu jami

# Alpine Linux
apk update && apk upgrade jami

# openSUSE
sudo zypper update jami

# FreeBSD
pkg update && pkg upgrade jami

# Always backup before updates
tar -czf /backup/jami-pre-update-$(date +%Y%m%d).tar.gz /etc/jami

# Restart after updates
sudo systemctl restart jami
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/jami -name "*.log" -mtime +30 -delete

# Verify integrity
sudo jami --verify || sudo jami check

# Update databases (if applicable)
sudo jami-update-db

# Optimize performance
sudo jami-optimize

# Check for security updates
sudo jami --security-check
```

## Additional Resources

- Official Documentation: https://docs.jami.org/
- GitHub Repository: https://github.com/jami/jami
- Community Forum: https://forum.jami.org/
- Wiki: https://wiki.jami.org/
- Comparison vs Jitsi, BigBlueButton, Element, Signal: https://docs.jami.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
