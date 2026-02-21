---
tags:
  - Docker
---

# Install Docker Engine on RHEL

To get started with Docker Engine on RHEL, ensure you meet the prerequisites and follow the installation steps.

## Prerequisites

### OS Requirements
To install Docker Engine, you need a maintained version of one of the following RHEL versions:
- RHEL 8
- RHEL 9

### Uninstall Old Versions
Before installing Docker Engine, uninstall any conflicting packages. Your Linux distribution may provide unofficial Docker packages that conflict with the official ones. Remove them using the following command:

```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
```

> **Note:** `dnf` might report that none of these packages are installed. Images, containers, volumes, and networks stored in `/var/lib/docker/` aren't automatically removed when you uninstall Docker.

---

## Installation Methods

You can install Docker Engine in different ways, depending on your needs:

1. **Using Docker's Repositories** (Recommended): Set up Docker's repositories for easier installation and upgrades.
2. **Manual RPM Installation**: Download and install the RPM package manually, useful for air-gapped systems.
3. **Automated Convenience Scripts**: Use scripts for quick installation in testing and development environments.

---

### Install Using the RPM Repository

#### Step 1: Set Up the Repository
Install the `dnf-plugins-core` package and add the Docker repository:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

#### Step 2: Install Docker Engine
To install the latest version:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify the GPG key fingerprint matches `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` before accepting it.

Start Docker and enable it to start on boot:

```bash
sudo systemctl enable --now docker
```

Verify the installation:

```bash
sudo docker run hello-world
```

> **Tip:** If you encounter permission errors, refer to the [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-root users to run Docker commands.

---

### Install from a Package

If you can't use Docker's repository, download the `.rpm` files for your RHEL version from [Docker's download page](https://download.docker.com/linux/rhel/). Install them using:

```bash
sudo dnf install ./containerd.io-<version>.<arch>.rpm \
  ./docker-ce-<version>.<arch>.rpm \
  ./docker-ce-cli-<version>.<arch>.rpm \
  ./docker-buildx-plugin-<version>.<arch>.rpm \
  ./docker-compose-plugin-<version>.<arch>.rpm
```

Start Docker:

```bash
sudo systemctl enable --now docker
```

Verify the installation:

```bash
sudo docker run hello-world
```

---

### Install Using the Convenience Script

Docker provides a script for non-interactive installation. Use it for development environments only:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> **Tip:** Use the `--dry-run` option to preview the script's steps before running it.

---

### Install Pre-Releases

To install pre-releases of Docker, use the test channel script:

```bash
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

---

## Upgrade Docker Engine

To upgrade Docker, follow the installation instructions for the desired version. If using manual RPM installation, replace `dnf install` with `dnf upgrade`.

---

## Uninstall Docker Engine

Remove Docker packages:

```bash
sudo dnf remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

To delete all images, containers, and volumes:

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

> **Note:** Manually delete any edited configuration files.

---

## Next Steps

Continue to [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/).
