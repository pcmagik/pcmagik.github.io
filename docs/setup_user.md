---
tags:
  - Linux
---

# Linux User Management

A guide covering user creation, password management, sudo configuration, and non-root Docker patterns.

---

## Creating Users

### Basic User Creation

```bash
# Create a user with home directory and bash shell
sudo useradd -m -s /bin/bash your_username

# Create a user with specific primary and supplementary groups
sudo useradd -g primary_group -G adm,docker,sudo -s /bin/bash your_username
```

**Flags explained:**

| Flag | Description |
|---|---|
| `-m` | Create home directory |
| `-s /bin/bash` | Set login shell |
| `-g group` | Set primary group |
| `-G grp1,grp2` | Set supplementary groups |
| `-u 1001` | Set specific UID |
| `-d /home/user` | Set custom home directory |

### Interactive User Creation

```bash
# Creates user interactively (prompts for password, info)
sudo adduser your_username
```

---

## Password Management

```bash
# Change another user's password (as root/sudo)
sudo passwd your_username

# Force password change on next login
sudo passwd -e your_username

# Lock / unlock a user account
sudo passwd -l your_username
sudo passwd -u your_username
```

---

## Shell Management

```bash
# Change user's shell
sudo chsh -s /bin/bash your_username

# List available shells
cat /etc/shells
```

---

## Sudo Configuration

### Add User to Sudo Group

```bash
# Add user to sudo group (Debian/Ubuntu)
sudo usermod -aG sudo your_username

# Add user to wheel group (RHEL/CentOS)
sudo usermod -aG wheel your_username
```

### Passwordless Sudo

```bash
# Create a sudoers drop-in file
echo "your_username ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/your_username
sudo chmod 0440 /etc/sudoers.d/your_username
```

> **Warning:** Passwordless sudo should only be used in development or automated environments. Always require passwords in production.

### Validate Sudoers Syntax

```bash
sudo visudo -c
```

---

## Group Management

```bash
# Create a new group
sudo groupadd mygroup

# Add user to a group
sudo usermod -aG mygroup your_username

# List user's groups
groups your_username
id your_username

# Remove user from a group
sudo gpasswd -d your_username mygroup
```

---

## Non-Root Docker Patterns

Running containers as non-root improves security. Below are common patterns.

### Set User in docker-compose.yml

```yaml
services:
  app:
    image: your_image:latest
    user: "1000:1000"
    volumes:
      - ./data:/app/data
```

### Pre-Create Directories with Correct Ownership

```bash
mkdir -p ./data
chown -R 1000:1000 ./data
```

### Custom Entrypoint for Permission Handling

**Dockerfile:**
```dockerfile
FROM node:20-slim

# Create non-root user
RUN groupadd -g 1000 appuser && \
    useradd -u 1000 -g appuser -m appuser

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["node", "app.js"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
# Ensure data directory has correct ownership
mkdir -p /app/data
chown -R 1000:1000 /app/data

exec "$@"
```

### Best Practices

- Always specify `USER` in Dockerfile when possible
- Map container UID/GID to a host user for volume mounts
- Create required directories on the host before starting containers
- Use `gosu` or `su-exec` for privilege dropping in entrypoints
