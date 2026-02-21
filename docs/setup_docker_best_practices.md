# Docker Best Practices

A collection of best practices for Docker including cleanup, multi-stage builds, non-root containers, security, logging, and monitoring.

---

## Docker Cleanup and Disk Management

### Remove Stopped Containers

```bash
docker container prune
```

### Remove Unused Images

```bash
# Dangling images only
docker image prune

# All unused images (not referenced by any container)
docker image prune -a
```

### Remove Unused Volumes

```bash
docker volume prune
```

### Remove Unused Networks

```bash
docker network prune
```

### Nuclear Option: Remove Everything Unused

```bash
docker system prune -a --volumes
```

> **Warning:** This removes all stopped containers, unused images, unused volumes, and unused networks. Make sure nothing important is left behind.

### Check Disk Usage

```bash
docker system df
docker system df -v  # verbose
```

### Stop and Remove All Containers

```bash
# Stop all running containers
docker container stop $(docker container ls -q)

# Remove all containers (including stopped)
docker container rm $(docker container ls -aq) --force
```

---

## Multi-Stage Builds

Multi-stage builds reduce final image size by separating build-time dependencies from runtime.

### Example: Node.js Application

```dockerfile
# Stage 1: Build
FROM node:20 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-slim

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**Benefits:**

- Build tools (compilers, dev dependencies) are not in the final image
- Smaller image = faster pulls, less storage, smaller attack surface
- Use `slim` or `alpine` base images for further size reduction

---

## Non-Root Containers

Running as root inside containers is a security risk. Always run as a non-root user.

### In Dockerfile

```dockerfile
FROM nginx:latest

# Create non-root user
RUN groupadd -g 1000 appuser && \
    useradd -u 1000 -g appuser -m appuser

# Set ownership
COPY --chown=appuser:appuser . /app

USER appuser
```

### In docker-compose.yml

```yaml
services:
  app:
    image: myapp:latest
    user: "1000:1000"
    volumes:
      - ./data:/app/data
```

### Pre-Create Directories with Correct Ownership

```bash
mkdir -p ./data
chown -R 1000:1000 ./data
```

---

## Docker Security Benchmarking

### Docker Bench Security

An automated script that checks Docker host and container configurations against CIS benchmarks:

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
./docker-bench-security.sh
```

This checks:
- Host configuration
- Docker daemon configuration
- Docker daemon configuration files
- Container images and build files
- Container runtime
- Docker security operations

---

## Log Management

### Docker Engine Log Locations

| Distribution | Log Location |
|---|---|
| Ubuntu (systemd) | `sudo journalctl -fu docker.service` |
| Debian | `/var/log/daemon.log` |
| CentOS / RHEL | `journalctl -u docker.service` |
| Fedora / OpenSUSE | `journalctl -u docker.service` |

### Container Logs

```bash
# View logs
docker logs <container_name>

# Follow logs in real time
docker logs -f <container_name>

# Last 100 lines
docker logs --tail 100 <container_name>

# Logs since a timestamp
docker logs --since "2024-01-01T00:00:00" <container_name>
```

### Log Driver Configuration

Configure log rotation in `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

```bash
sudo systemctl restart docker
```

---

## Monitoring Tools

### ctop (Container Top)

Interactive container metrics viewer:

```bash
# Install via apt
sudo apt-get install ca-certificates curl gnupg lsb-release
curl -fsSL https://azlux.fr/repo.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/azlux-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/azlux-archive-keyring.gpg] http://packages.azlux.fr/debian $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/azlux.list
sudo apt-get update && sudo apt-get install docker-ctop

# Or install manually
sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.7/ctop-0.7.7-linux-amd64 -O /usr/local/bin/ctop
sudo chmod +x /usr/local/bin/ctop
```

### lazydocker

Terminal UI for Docker management:

```bash
curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash
```

### docker stats

Built-in resource monitoring:

```bash
docker stats
docker stats --no-stream  # snapshot
```

---

## DNS Configuration

If containers cannot resolve DNS names, configure Docker DNS:

Edit `/etc/docker/daemon.json`:

```json
{
  "dns": ["1.1.1.1", "8.8.8.8"]
}
```

```bash
sudo systemctl restart docker
```
