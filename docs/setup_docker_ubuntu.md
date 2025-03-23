# Install Docker Engine on Ubuntu

To get started with Docker Engine on Ubuntu, ensure you meet the prerequisites and follow the installation steps.

## Prerequisites

### Firewall Limitations

**Warning**: Before installing Docker, consider the following security implications and firewall incompatibilities:

- If you use `ufw` or `firewalld` to manage firewall settings, be aware that exposing container ports using Docker bypasses your firewall rules. For more information, refer to [Docker and ufw](https://docs.docker.com/network/iptables/#docker-and-ufw).
- Docker is only compatible with `iptables-nft` and `iptables-legacy`. Firewall rules created with `nft` are not supported. Ensure your firewall rulesets are created with `iptables` or `ip6tables` and added to the `DOCKER-USER` chain. See [Packet filtering and firewalls](https://docs.docker.com/network/iptables/#packet-filtering-and-firewalls).

### OS Requirements

To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:

- Ubuntu Oracular 24.10
- Ubuntu Noble 24.04 (LTS)
- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Focal 20.04 (LTS)

Docker Engine for Ubuntu is compatible with the following architectures: `x86_64` (or `amd64`), `armhf`, `arm64`, `s390x`, and `ppc64le` (ppc64el).

> **Note**: Installation on Ubuntu derivative distributions, such as Linux Mint, is not officially supported (though it may work).

## Uninstall Old Versions

Before installing Docker Engine, uninstall any conflicting packages. Your Linux distribution may provide unofficial Docker packages that conflict with the official packages provided by Docker. Uninstall these packages before proceeding.

### Unofficial Packages to Uninstall

- `docker.io`
- `docker-compose`
- `docker-compose-v2`
- `docker-doc`
- `podman-docker`

Docker Engine depends on `containerd` and `runc`, which are bundled as `containerd.io`. If you have previously installed `containerd` or `runc`, uninstall them to avoid conflicts.

Run the following command to uninstall all conflicting packages:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

> **Note**: `apt-get` might report that none of these packages are installed.

Images, containers, volumes, and networks stored in `/var/lib/docker/` aren't automatically removed when you uninstall Docker. To start with a clean installation, refer to the [uninstall Docker Engine](https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine) section.

## Installation Methods

You can install Docker Engine in different ways, depending on your needs:

1. **Docker Desktop for Linux**: Bundles Docker Engine and is the easiest way to get started.
2. **Using Docker's apt repository**: Set up and install Docker Engine from Docker's repository.
3. **Manual installation**: Download and install `.deb` packages manually.
4. **Convenience script**: Recommended only for testing and development environments.

### Install Using the apt Repository

Before installing Docker Engine for the first time on a new host machine, set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

#### Set Up Docker's apt Repository

1. Add Docker's official GPG key:

  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc
  ```

2. Add the repository to Apt sources:

  ```bash
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  ```

#### Install Docker Packages

To install the latest version of Docker Engine, run:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify the installation by running the `hello-world` image:

```bash
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

> **Tip**: If you encounter errors when trying to run Docker without `sudo`, refer to the [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands.

### Upgrade Docker Engine

To upgrade Docker Engine, follow the installation instructions and choose the new version you want to install.

### Install Using the Convenience Script

Docker provides a convenience script at [https://get.docker.com/](https://get.docker.com/) to install Docker non-interactively. This script is useful for development environments but not recommended for production.

#### Example

Download and run the script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> **Tip**: Use the `--dry-run` option to preview the script steps before running it.

### Uninstall Docker Engine

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

You must manually delete any edited configuration files.

## Next Steps

Continue to [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/).
