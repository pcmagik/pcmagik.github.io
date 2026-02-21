---
tags:
  - Linux
---

# Linux Administration Essentials

A quick-reference guide covering the most common Linux administration tasks for homelab and server environments.

---

## System Update and Package Management

### Ubuntu / Debian (APT)

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo apt-get autoremove -y
```

> **Tip:** Use `apt list --upgradable` to preview available updates before upgrading.

### RHEL / CentOS / Fedora (DNF)

```bash
sudo dnf check-update
sudo dnf upgrade -y
sudo dnf autoremove -y
```

### Install Common Tools

```bash
sudo apt update && sudo apt install -y \
  mc nano net-tools iputils-ping curl wget git htop \
  tcpdump traceroute vim zip unzip neofetch bash-completion
```

---

## Service Management (systemd)

```bash
# Check service status
sudo systemctl status <service_name>

# Start / stop / restart a service
sudo systemctl start <service_name>
sudo systemctl stop <service_name>
sudo systemctl restart <service_name>

# Enable service at boot
sudo systemctl enable <service_name>

# View service logs
sudo journalctl -u <service_name> -f
sudo journalctl -u <service_name> --since "1 hour ago"
```

---

## Text Editors Quick Reference

### nano

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Save file |
| `Ctrl+X` | Exit |
| `Ctrl+W` | Search |
| `Ctrl+K` | Cut line |
| `Ctrl+U` | Paste line |
| `Ctrl+G` | Help |
| `Ctrl+C` | Show cursor position |
| `Alt+U` | Undo |

### vim

| Command | Action |
|---|---|
| `i` | Enter insert mode |
| `Esc` | Exit insert mode |
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |
| `:set number` | Show line numbers |
| `/pattern` | Search forward |

---

## Network Configuration with Netplan

Netplan is the default network configuration tool on Ubuntu 18.04+.

### Configuration file location

```bash
ls /etc/netplan/
# Typical files: 00-installer-config.yaml, 50-cloud-init.yaml
```

### DHCP Configuration

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
```

### Static IP Configuration

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 10.0.0.100/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 1.0.0.1
```

### Apply Changes

```bash
sudo netplan apply

# Debug mode (verbose output)
sudo netplan --debug apply
```

### Disable Cloud-Init Network Config

If cloud-init keeps overwriting your netplan config:

```bash
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

---

## Firewall Management

### UFW (Uncomplicated Firewall)

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh
sudo ufw allow 22/tcp

# Allow specific port
sudo ufw allow 8080/tcp

# Deny a port
sudo ufw deny 3306/tcp

# Check status
sudo ufw status verbose

# Disable UFW
sudo ufw disable
```

### IPTables Basics

```bash
# List all rules
sudo iptables -L -v

# Flush all rules (reset)
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F

# Set default policies
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Save rules persistently
sudo apt-get install iptables-persistent
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# Restore saved rules
sudo iptables-restore < /etc/iptables/rules.v4

# NAT / Masquerade
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

---

## LVM Resize

### Remove local-lvm and Extend Root

Useful on Proxmox or similar setups where `local-lvm` is unused:

```bash
# Remove the data logical volume
lvremove /dev/pve/data

# Option A: Give 50% free space to root, keep rest for new LV
lvresize -l +50%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root

# Option B: Give ALL free space to root
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

### Recreate a Thin Pool (if needed)

```bash
lvcreate -l 100%FREE --poolmetadatasize 1024M --chunksize 256 -T -n data pve
```

---

## Password Generation and Hashing

### Install Required Tools

```bash
sudo apt update && sudo apt install -y apache2-utils
```

### Generate Hashed Password (htpasswd)

```bash
# Generate bcrypt hash
htpasswd -nbBC 10 "" "your_password_here" | tr -d ':\n'

# Generate APR1 hash (for traefik, nginx basic auth)
echo $(htpasswd -nb "your_username" "your_password_here") | sed -e s/\\$/\\$\\$/g
```

### Generate Random Password

```bash
# Using openssl
openssl rand -base64 24

# Using /dev/urandom
tr -dc 'A-Za-z0-9!@#$%' < /dev/urandom | head -c 32; echo
```

---

## Diagnostic Tools

```bash
# System resources
htop                          # Interactive process viewer
free -h                       # Memory usage
df -h                         # Disk usage
du -sh /path/to/dir           # Directory size

# Network diagnostics
ip addr show                  # Show IP addresses
ip route show                 # Show routing table
ping -c 4 10.0.0.1            # Test connectivity
traceroute 10.0.0.1           # Trace network path
tcpdump -i eth0 port 80       # Capture packets
ss -tulnp                     # Show listening ports
curl -4 icanhazip.com         # Get public IP

# System information
uname -a                      # Kernel info
neofetch                      # System summary
lsblk                         # Block devices
lscpu                         # CPU info
```

---

## SSL Certificate Generation (Self-Signed)

```bash
sudo openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 -sha256 \
  -keyout /etc/ssl/private/selfsigned.key \
  -out /etc/ssl/certs/selfsigned.crt \
  -subj "/CN=server.example.com" \
  -addext "subjectAltName=DNS:server.example.com"
```
