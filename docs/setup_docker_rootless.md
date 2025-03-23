# Rootless Mode

Rootless mode allows running the Docker daemon and containers as a non-root user to mitigate potential vulnerabilities in the daemon and the container runtime. It does not require root privileges during installation, provided the prerequisites are met.

## How It Works

Rootless mode executes the Docker daemon and containers inside a user namespace. Unlike userns-remap mode, where the daemon runs with root privileges, in rootless mode, both the daemon and containers run without root privileges. It avoids using binaries with SETUID bits or file capabilities, except for `newuidmap` and `newgidmap`, which enable multiple UIDs/GIDs in the user namespace.

## Prerequisites

1. Install `newuidmap` and `newgidmap` (provided by the `uidmap` package on most distributions).
2. Ensure `/etc/subuid` and `/etc/subgid` contain at least 65,536 subordinate UIDs/GIDs for the user.

Example:
```bash
id -u
1001
whoami
testuser
grep ^$(whoami): /etc/subuid
testuser:231072:65536
grep ^$(whoami): /etc/subgid
testuser:231072:65536
```

## Distribution-Specific Hints

### Ubuntu
- Install `dbus-user-session` and `uidmap` packages:
    ```bash
    sudo apt-get install -y dbus-user-session uidmap
    ```
- If using a terminal where the user was not directly logged in, install `systemd-container`:
    ```bash
    sudo apt-get install -y systemd-container
    sudo machinectl shell TheUser@
    ```
- For Ubuntu 24.04 and later, configure AppArmor for unprivileged user namespaces if needed.

### Other Distributions
Refer to the specific instructions for Debian, Arch Linux, openSUSE, CentOS, RHEL, and Fedora.

## Known Limitations

- **Supported storage drivers**:
  - `overlay2` (kernel 5.11+ or Ubuntu-flavored kernel)
  - `fuse-overlayfs` (kernel 4.18+)
  - `btrfs` (with `user_subvol_rm_allowed` mount option)
  - `vfs`
- **Cgroup v2 and systemd** are required for resource limiting.
- **Unsupported features**:
  - AppArmor
  - Checkpoint
  - Overlay network
  - Exposing SCTP ports
- **Networking limitations**:
  - IP address in `docker inspect` is namespaced.
  - Host network (`--net=host`) is namespaced.
  - NFS mounts as `data-root` are not supported.

## Installation

### With Packages (RPM/DEB)
1. Run the setup tool as a non-root user:
    ```bash
    dockerd-rootless-setuptool.sh install
    ```
2. Add the following to `~/.bashrc`:
    ```bash
    export PATH=/usr/bin:$PATH
    export DOCKER_HOST=unix:///run/user/1000/docker.sock
    ```

### Without Packages
Install `docker-ce-rootless-extras` manually:
```bash
sudo apt-get install -y docker-ce-rootless-extras
```

If the system-wide Docker daemon is already running, consider disabling it:
```bash
sudo systemctl disable --now docker.service docker.socket
sudo rm /var/run/docker.sock
```

Run the setup tool:
```bash
dockerd-rootless-setuptool.sh install
```

Add the following to `~/.bashrc`:
```bash
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```

## Uninstallation

To remove the systemd service of the Docker daemon:
```bash
dockerd-rootless-setuptool.sh uninstall
```

Unset environment variables `PATH` and `DOCKER_HOST` if added to `~/.bashrc`. To remove data:
```bash
rootlesskit rm -rf ~/.local/share/docker
```

To remove binaries, delete the `docker-ce-rootless-extras` package or manually remove binaries under `~/bin`.

## Usage

### Daemon Management
With systemd:
```bash
systemctl --user start docker
systemctl --user enable docker
sudo loginctl enable-linger $(whoami)
```

### Client Configuration
Specify the socket path:
```bash
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
docker run -d -p 8080:80 nginx
```

Or use CLI context:
```bash
docker context use rootless
docker run -d -p 8080:80 nginx
```

## Best Practices

### Running Rootless Docker in Docker
Use the `docker:<version>-dind-rootless` image:
```bash
docker run -d --name dind-rootless --privileged docker:25.0-dind-rootless
```

### Exposing Docker API Socket
Expose through TCP:
```bash
DOCKERD_ROOTLESS_ROOTLESSKIT_FLAGS="-p 0.0.0.0:2376:2376/tcp" \
dockerd-rootless.sh -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem
```

Expose through SSH:
```bash
ssh -l REMOTEUSER REMOTEHOST 'echo $DOCKER_HOST'
docker -H ssh://REMOTEUSER@REMOTEHOST run ...
```

### Resource Limiting
Use cgroup v2 and systemd for resource limiting:
```bash
mkdir -p /etc/systemd/system/user@.service.d
cat > /etc/systemd/system/user@.service.d/delegate.conf << EOF
[Service]
Delegate=cpu cpuset io memory pids
EOF
systemctl daemon-reload
```

## Troubleshooting

### Common Errors
- **Systemd not detected**: Use `dockerd-rootless.sh` manually.
- **Unprivileged user namespace disabled**: Set `kernel.unprivileged_userns_clone=1` in `/etc/sysctl.conf`.
- **Insufficient subuid/subgid ranges**: Configure `/etc/subuid` and `/etc/subgid`.

### Networking Issues
- **Ping doesn't work**: Add `net.ipv4.ping_group_range = 0 2147483647` to `/etc/sysctl.conf`.
- **Slow network**: Install `slirp4netns` or adjust MTU.

For more details, refer to the official Docker documentation.
```
