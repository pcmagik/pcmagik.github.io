---
tags:
  - Docker
---

# Manage Docker as a Non-Root User

The Docker daemon binds to a Unix socket, not a TCP port. By default, the root user owns the Unix socket, and other users can only access it using `sudo`. The Docker daemon always runs as the root user.

If you don't want to preface the `docker` command with `sudo`, create a Unix group called `docker` and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the `docker` group. On some Linux distributions, the system automatically creates this group when installing Docker Engine using a package manager. In that case, there is no need to manually create the group.

> **Warning**  
> The `docker` group grants root-level privileges to the user. For details on how this impacts security in your system, see [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/#docker-daemon-attack-surface).

**Note**: To run Docker without root privileges, see [Run the Docker daemon as a non-root user (Rootless mode)](https://docs.docker.com/engine/security/rootless/).

## Steps to Create the Docker Group and Add Your User

1. **Create the docker group**:
    ```bash
    sudo groupadd docker
    ```

2. **Add your user to the docker group**:
    ```bash
    sudo usermod -aG docker $USER
    ```

3. **Log out and log back in** so that your group membership is re-evaluated.

    - If you're running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.
    - Alternatively, run the following command to activate the changes to groups:
      ```bash
      newgrp docker
      ```

4. **Verify that you can run Docker commands without sudo**:
    ```bash
    docker run hello-world
    ```
    This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

### Fixing Permission Errors

If you initially ran Docker CLI commands using `sudo` before adding your user to the `docker` group, you may encounter the following error:
```
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

To fix this issue, either:

- Remove the `~/.docker/` directory (it will be recreated automatically, but any custom settings will be lost):
  ```bash
  rm -rf ~/.docker
  ```
- Or change its ownership and permissions:
  ```bash
  sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
  sudo chmod g+rwx "$HOME/.docker" -R
  ```

## Configure Docker to Start on Boot with systemd

Many modern Linux distributions use `systemd` to manage services at boot. On Debian and Ubuntu, the Docker service starts on boot by default. To enable this behavior on other Linux distributions using `systemd`, run:
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To disable this behavior, use:
```bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

You can also use `systemd` unit files to configure the Docker service on startup. For example, you can add an HTTP proxy, set a different directory for Docker runtime files, or apply other customizations. See [Configure the daemon to use a proxy](https://docs.docker.com/config/daemon/systemd/#http-proxy) for more details.

## Configure Default Logging Driver

Docker provides logging drivers for collecting and viewing log data from all containers running on a host. The default logging driver, `json-file`, writes log data to JSON-formatted files on the host filesystem. Over time, these log files can grow in size, potentially exhausting disk resources.

To avoid disk overuse, consider one of the following options:

- Configure the `json-file` logging driver to enable log rotation.
- Use an alternative logging driver, such as the `local` logging driver, which performs log rotation by default.
- Use a logging driver that sends logs to a remote logging aggregator.

## Next Steps

Explore the [Docker workshop](https://docs.docker.com/get-started/) to learn how to build an image and run it as a containerized application.