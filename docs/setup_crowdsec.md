---
tags:
  - Security
---

# CrowdSec Setup Guide

A guide for installing and configuring CrowdSec as a collaborative intrusion prevention system with firewall bouncer integration.

---

## Installation

### Install CrowdSec

```bash
curl -s https://install.crowdsec.net | sudo sh
sudo apt install crowdsec
```

### Install Firewall Bouncer

```bash
sudo apt install crowdsec-firewall-bouncer-iptables
```

### Start and Enable Services

```bash
sudo systemctl restart crowdsec
sudo systemctl enable crowdsec
sudo systemctl start crowdsec-firewall-bouncer
sudo systemctl enable crowdsec-firewall-bouncer
```

---

## Parsers and Collections

Install parsers to analyze log formats and collections for specific services:

### Log Parsers

```bash
cscli parsers install crowdsecurity/nginx-logs
cscli parsers install crowdsecurity/http-logs
cscli parsers install crowdsecurity/apache2-logs
```

### Collections

```bash
cscli collections install crowdsecurity/linux
cscli collections install crowdsecurity/docker
cscli collections install crowdsecurity/nginx
```

### Scenarios

Scenarios define attack patterns to detect:

```bash
cscli scenarios install crowdsecurity/http-crawl-non_statics
cscli scenarios install crowdsecurity/http-probing
cscli scenarios install crowdsecurity/http-bad-user-agent
cscli scenarios install crowdsecurity/http-path-traversal-probing
```

---

## Firewall Bouncer Configuration

### Configuration File

Location: `/etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml`

```yaml
mode: iptables
update_frequency: 10s
log_mode: file
log_dir: /var/log/
log_level: info
log_compression: true
log_max_size: 100
log_max_backups: 3
log_max_age: 30
api_url: http://127.0.0.1:8080/
api_key: your_api_token_here
insecure_skip_verify: false
disable_ipv6: false
deny_action: DROP
deny_log: false
supported_decisions_types:
  - ban
blacklists_ipv4: crowdsec-blacklists
blacklists_ipv6: crowdsec6-blacklists
ipset_type: nethash
iptables_chains:
  - INPUT
#  - FORWARD
#  - DOCKER-USER
```

### nftables Mode

If using nftables instead of iptables, set `mode: nftables` and configure:

```yaml
nftables:
  ipv4:
    enabled: true
    set-only: false
    table: crowdsec
    chain: crowdsec-chain
    priority: -10
  ipv6:
    enabled: true
    set-only: false
    table: crowdsec6
    chain: crowdsec6-chain
    priority: -10

nftables_hooks:
  - input
  - forward
```

---

## API Key Management

### Generate a New Bouncer Key

```bash
sudo cscli bouncers add my-bouncer --output yaml
```

### Test API Connection

```bash
curl -H "X-Api-Key: your_api_token_here" http://127.0.0.1:8080/v1/alerts
```

### Regenerate Key If Needed

```bash
sudo cscli bouncers delete my-bouncer
sudo cscli bouncers add my-bouncer --output yaml
```

Update the key in the bouncer YAML and restart:

```bash
sudo systemctl restart crowdsec-firewall-bouncer
```

---

## Monitoring

### View Metrics

```bash
cscli metrics
```

### List Active Decisions (Bans)

```bash
cscli decisions list
```

### List Alerts

```bash
cscli alerts list
```

### Check Bouncer Status

```bash
sudo systemctl status crowdsec-firewall-bouncer
```

---

## Updating

### Update Hub (Parsers, Scenarios, Collections)

```bash
cscli hub update
cscli hub upgrade
```

### Update CrowdSec

```bash
sudo apt update && sudo apt upgrade crowdsec crowdsec-firewall-bouncer-iptables
```

---

## Permissions Setup

### Add User to CrowdSec Group

```bash
sudo groupadd crowdsec 2>/dev/null
sudo usermod -aG crowdsec $USER
```

> **Note:** Log out and back in for group changes to take effect.

### Passwordless cscli via Sudo

```bash
echo "$USER ALL=(ALL) NOPASSWD: /usr/bin/cscli" | sudo tee /etc/sudoers.d/cscli
```

### Reload Configuration

```bash
sudo systemctl reload crowdsec
```
