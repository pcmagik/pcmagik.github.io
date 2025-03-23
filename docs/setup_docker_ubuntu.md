# Docker Installation Guide

## Install Docker on Ubuntu

### OS Requirements
To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:

- Ubuntu Oracular 24.10  
- Ubuntu Noble 24.04 (LTS)  
- Ubuntu Jammy 22.04 (LTS)  
- Ubuntu Focal 20.04 (LTS)  

---

## Manual Installation Steps

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
- To install the latest version:
  ```bash
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

- Verify the installation:
  ```bash
  sudo docker run hello-world
  ```
  This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

---

## Automated Installation Script

### Download the Docker Installation Script
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

### Preview Script Steps Before Running
Run the script with the `--dry-run` option to preview the steps:
```bash
sudo sh ./get-docker.sh --dry-run
```

### Run the Installation Script
```bash
sudo sh ./get-docker.sh
```


### Commnads to Check Docker Installation

### Commands to Manage Docker Service

For the older OS versions or WSL:
```bash
sudo service docker start
sudo service docker status
sudo service docker restart
sudo service docker stop
sudo service docker enable
sudo service docker disable
```

For the newer OS versions:
```bash
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl restart docker
sudo systemctl stop docker
sudo systemctl enable docker
sudo systemctl disable docker
```
```
### Verify Docker Installation
```bash
sudo docker --version
sudo docker-compose --version
sudo docker info
sudo docker version
sudo docker images
sudo docker ps -a
sudo docker network ls
sudo docker volume ls
sudo docker container ls -a
sudo docker run hello-world
```
### Run a Test Container
```bash

sudo docker run hello-world









---

## Rootless Docker Installation

### Install Rootless Docker
```bash
dockerd-rootless-setuptool.sh install
```

#### If `dockerd-rootless-setuptool.sh` is Missing
Install the `docker-ce-rootless-extras` package manually:
```bash
sudo apt-get install -y docker-ce-rootless-extras
```

---

## Uninstall Docker Engine

### Remove Docker Engine, CLI, and Related Packages
```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

### Remove Images, Containers, and Volumes
To delete all images, containers, and volumes:
```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

### Remove Source List and Keyrings
```bash
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```

**Note:** You must manually delete any edited configuration files.