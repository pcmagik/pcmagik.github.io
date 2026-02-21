---
tags:
  - Docker
---

# Install Docker Engine on Debian

To get started with Docker Engine on Debian, make sure you meet the prerequisites, and then follow the installation steps.

## Prerequisites

### Firewall Limitations

**Warning**: Before you install Docker, consider the following security implications and firewall incompatibilities:

- If you use `ufw` or `firewalld` to manage firewall settings, be aware that when you expose container ports using Docker, these ports bypass your firewall rules. For more information, refer to [Docker and ufw](https://docs.docker.com/network/iptables/).
- Docker is only compatible with `iptables-nft` and `iptables-legacy`. Firewall rules created with `nft` are not supported on a system with Docker installed. Ensure that any firewall rulesets you use are created with `iptables` or `ip6tables`, and that you add them to the `DOCKER-USER` chain. See [Packet filtering and firewalls](https://docs.docker.com/network/iptables/#docker-user-defined-chains).

### OS Requirements

To install Docker Engine, you need the 64-bit version of one of these Debian versions:

- **Debian Bookworm 12 (stable)**
- **Debian Bullseye 11 (oldstable)**

Docker Engine for Debian is compatible with the following architectures:

- `x86_64` (or `amd64`)
- `armhf`
- `arm64`
- `ppc64le` (or `ppc64el`)

### Uninstall Old Versions

Before installing Docker Engine, uninstall any conflicting packages. Your Linux distribution may provide unofficial Docker packages, which may conflict with the official packages provided by Docker. Uninstall these packages before proceeding:

- `docker.io`
- `docker-compose`
- `docker-doc`
- `podman-docker`

Docker Engine depends on `containerd` and `runc`, which are bundled as `containerd.io`. If you have previously installed `containerd` or `runc`, uninstall them to avoid conflicts.

Run the following command to uninstall all conflicting packages:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

> **Note**: `apt-get` might report that none of these packages are installed.

Images, containers, volumes, and networks stored in `/var/lib/docker/` aren't automatically removed when you uninstall Docker. If you want a clean installation, refer to the [uninstall Docker Engine](https://docs.docker.com/engine/install/linux-postinstall/) section.

---

## Installation Methods

You can install Docker Engine in different ways, depending on your needs:

1. **Docker Desktop for Linux**: The easiest and quickest way to get started.
2. **Set up and install Docker Engine from Docker's apt repository**.
3. **Install manually and manage upgrades manually**.
4. **Use a convenience script**: Recommended only for testing and development environments.

---

## Install Using the apt Repository

Before installing Docker Engine for the first time on a new host machine, set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

### Set Up Docker's apt Repository

1. Add Docker's official GPG key:

  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc
  ```

2. Add the repository to Apt sources:

  ```bash
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  ```

  > **Note**: If you use a derivative distribution, such as Kali Linux, you may need to substitute the part of this command that prints the version codename:
  >
  > ```bash
  > (. /etc/os-release && echo "$VERSION_CODENAME")
  > ```
  >
  > Replace this part with the codename of the corresponding Debian release, such as `bookworm`.

### Install the Docker Packages

#### Latest Version

To install the latest version, run:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify the installation by running the `hello-world` image:

```bash
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

> **Tip**: Receiving errors when trying to run without root? The `docker` user group exists but contains no users, which is why you’re required to use `sudo` to run Docker commands. Refer to [Linux postinstall](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands.

---

## Upgrade Docker Engine

To upgrade Docker Engine, follow the installation instructions, choosing the new version you want to install.

---

## Install Using the Convenience Script

Docker provides a convenience script at [https://get.docker.com/](https://get.docker.com/) to install Docker into development environments non-interactively. This script isn't recommended for production environments but is useful for creating a provisioning script tailored to your needs.

### Example

Download and run the script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> **Tip**: Preview script steps before running by using the `--dry-run` option:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

---

## Uninstall Docker Engine

To uninstall Docker Engine, CLI, `containerd`, and Docker Compose packages, run:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

To delete all images, containers, and volumes:

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

Remove source list and keyrings:

```bash
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```

> **Note**: You must delete any edited configuration files manually.

---

## Next Steps

Continue to [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/).
