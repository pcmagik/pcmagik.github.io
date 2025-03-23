# Install Docker Engine on Debian

## OS Requirements

To install Docker Engine, you need the 64-bit version of one of these Debian versions:

- **Debian Bookworm 12 (stable)**
- **Debian Bullseye 11 (oldstable)**

Docker Engine for Debian is compatible with the following architectures:
- `x86_64` (or `amd64`)
- `armhf`
- `arm64`
- `ppc64le` (or `ppc64el`)

## Installation Methods

You can install Docker Engine in different ways, depending on your needs:

1. **Docker Desktop for Linux**: Comes bundled with Docker Engine. This is the easiest and quickest way to get started.
2. **Docker's apt repository**: Set up and install Docker Engine from Docker's apt repository.
3. **Manual installation**: Install and manage upgrades manually.
4. **Convenience script**: Recommended only for testing and development environments.

---

## Install Using the apt Repository

Before installing Docker Engine for the first time on a new host machine, set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

### Step 1: Set Up Docker's apt Repository

1. **Add Docker's official GPG key:**
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```

2. **Add the repository to Apt sources:**
    ```bash
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```

> **Note**:  
> If you use a derivative distribution, such as Kali Linux, you may need to substitute the part of this command that prints the version codename:  
> `(. /etc/os-release && echo "$VERSION_CODENAME")`  
> Replace this part with the codename of the corresponding Debian release, such as `bookworm`.

---

### Step 2: Install the Docker Packages

- **To install the latest version**, run:
  ```bash
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

- **Verify the installation** by running the `hello-world` image:
  ```bash
  sudo docker run hello-world
  ```
  This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

---

### Preview Script Steps Before Running

You can preview the steps of the installation script using the `--dry-run` option:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

---

### Use Docker as a Non-Privileged User or Install in Rootless Mode

The installation script requires root or `sudo` privileges to install and use Docker. If you want to grant non-root users access to Docker, refer to the [post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/).

You can also install Docker without root privileges or configure it to run in rootless mode. For instructions, refer to [Run the Docker Daemon as a Non-Root User (Rootless Mode)](https://docs.docker.com/engine/security/rootless/).