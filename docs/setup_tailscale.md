---
tags:
  - Security
  - VPN
---

# Tailscale Setup Guide

A guide for installing and configuring Tailscale as a mesh VPN, including subnet routing, exit nodes, and Docker integration.

---

## Installation

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

---

## Basic Setup

### Start Tailscale

```bash
sudo tailscale up
```

### Check Status

```bash
tailscale status
```

### Authenticate

Follow the printed URL to authenticate the device in the Tailscale admin console at [https://login.tailscale.com](https://login.tailscale.com).

---

## Subnet Router

Advertise local network routes so other Tailscale nodes can reach devices on this network.

### Enable IP Forwarding

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

### Advertise Routes

```bash
sudo tailscale up --advertise-routes=10.0.0.0/24 --accept-routes
```

> **Important:** After advertising routes, approve them in the Tailscale admin console under the machine's route settings.

---

## Exit Node

Allow other Tailscale devices to route all internet traffic through this machine.

```bash
sudo tailscale up --advertise-routes=10.0.0.0/24 --accept-routes --advertise-exit-node
```

Approve the exit node in the Tailscale admin console.

### Use an Exit Node (Client Side)

```bash
sudo tailscale up --exit-node=<exit_node_ip>
```

---

## UDP GRO Forwarding (Performance)

Optimize network performance for forwarding:

```bash
NETDEV=$(ip route show 0/0 | awk 'NR==1 {print $5}')
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on
```

### Make Persistent via NetworkD Dispatcher

```bash
cat << 'EOF' | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
#!/bin/sh
NETDEV=$(ip route show 0/0 | awk 'NR==1 {print $5}')
ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro off
EOF

sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale
```

---

## Docker + Tailscale Integration

### Problem: Stateful Filtering Warning

When running Docker alongside Tailscale, you may see:

```
Warning: Stateful filtering is enabled and Docker was detected;
this may prevent Docker containers on this host from connecting to Tailscale nodes.
```

### Solution 1: IPTables Rules (Recommended)

```bash
sudo iptables -I DOCKER-USER -i tailscale0 -j ACCEPT
sudo iptables -I DOCKER-USER -o tailscale0 -j ACCEPT
sudo iptables -A FORWARD -i docker0 -o tailscale0 -j ACCEPT
sudo iptables -A FORWARD -i tailscale0 -o docker0 -j ACCEPT

# Save rules persistently
sudo apt install -y iptables-persistent
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

### Solution 2: Docker Daemon Configuration

Edit `/etc/docker/daemon.json`:

```json
{
  "dns": ["1.1.1.1", "8.8.8.8"],
  "iptables": false,
  "bridge": "none"
}
```

```bash
sudo systemctl restart docker
```

> **Warning:** Disabling Docker's iptables management means you lose Docker's built-in network isolation. Use Solution 1 unless you have specific reasons.

---

## Auto-Update

```bash
sudo tailscale set --auto-update
```

---

## Troubleshooting

### Restart Tailscale

```bash
sudo systemctl restart tailscaled
tailscale status
```

### Check Daemon Logs

```bash
sudo journalctl -u tailscaled -f
```

### Subnet Routing Issues

If subnet routes are not working:

1. Verify IP forwarding is enabled:
```bash
sysctl net.ipv4.ip_forward
```

2. Verify routes are approved in admin console

3. Try with explicit flags:
```bash
sudo tailscale up \
  --advertise-routes=10.0.0.0/24 \
  --accept-routes \
  --snat-subnet-routes=false
```

### Docker Not Restarting

If Docker fails to restart after editing `daemon.json`:

```bash
# Check for JSON syntax errors
python3 -c "import json; json.load(open('/etc/docker/daemon.json'))"

# Check Docker logs
sudo journalctl -xeu docker.service
```
