# Install Docker Engine on RHEL

To get started with Docker Engine on RHEL, make sure you meet the prerequisites, and then follow the installation steps.

## Prerequisites

### OS Requirements

To install Docker Engine, you need a maintained version of one of the following RHEL versions:

- RHEL 8
- RHEL 9

## Set Up the Repository

Install the `dnf-plugins-core` package (which provides the commands to manage your DNF repositories) and set up the repository:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

## Install Docker Engine

### Install the Docker Packages

#### Latest Version

To install the latest version, run:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

If prompted to accept the GPG key, verify that the fingerprint matches `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`, and if so, accept it.

This command installs Docker, but it doesn't start Docker. It also creates a `docker` group; however, it doesn't add any users to the group by default.

### Start Docker Engine

Start Docker and configure it to start automatically on boot:

```bash
sudo systemctl enable --now docker
```

If you don't want Docker to start automatically, use:

```bash
sudo systemctl start docker
```

### Verify the Installation

Verify that the installation is successful by running the `hello-world` image:

```bash
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.