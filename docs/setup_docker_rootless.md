```markdown
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

- Supported storage drivers:
    - `overlay2` (kernel 5.11+ or Ubuntu-flavored kernel)
    - `fuse-overlayfs` (kernel 4.18+)
    - `btrfs` (with `user_subvol_rm_allowed` mount option)
    - `vfs`
- Cgroup v2 and systemd are required for resource limiting.
- Unsupported features:
    - AppArmor
    - Checkpoint
    - Overlay network
    - Exposing SCTP ports
- Networking limitations:
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


 sudo systemctl disable --now docker.service docker.socket
 sudo rm /var/run/docker.sock
Should you choose not to shut down the docker service and socket, you will need to use the --force parameter in the next section. There are no known issues, but until you shutdown and disable you're still running rootful Docker.

With packages (RPM/DEB) Without packages
If you installed Docker 20.10 or later with RPM/DEB packages, you should have dockerd-rootless-setuptool.sh in /usr/bin.

Run dockerd-rootless-setuptool.sh install as a non-root user to set up the daemon:


 dockerd-rootless-setuptool.sh install
[INFO] Creating /home/testuser/.config/systemd/user/docker.service
...
[INFO] Installed docker.service successfully.
[INFO] To control docker.service, run: `systemctl --user (start|stop|restart) docker.service`
[INFO] To run docker.service on system startup, run: `sudo loginctl enable-linger testuser`

[INFO] Make sure the following environment variables are set (or add them to ~/.bashrc):

export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
If dockerd-rootless-setuptool.sh is not present, you may need to install the docker-ce-rootless-extras package manually, e.g.,


 sudo apt-get install -y docker-ce-rootless-extras
See Troubleshooting if you faced an error.

Uninstall
To remove the systemd service of the Docker daemon, run dockerd-rootless-setuptool.sh uninstall:


 dockerd-rootless-setuptool.sh uninstall
+ systemctl --user stop docker.service
+ systemctl --user disable docker.service
Removed /home/testuser/.config/systemd/user/default.target.wants/docker.service.
[INFO] Uninstalled docker.service
[INFO] This uninstallation tool does NOT remove Docker binaries and data.
[INFO] To remove data, run: `/usr/bin/rootlesskit rm -rf /home/testuser/.local/share/docker`
Unset environment variables PATH and DOCKER_HOST if you have added them to ~/.bashrc.

To remove the data directory, run rootlesskit rm -rf ~/.local/share/docker.

To remove the binaries, remove docker-ce-rootless-extras package if you installed Docker with package managers. If you installed Docker with https://get.docker.com/rootless ( Install without packages), remove the binary files under ~/bin:


 cd ~/bin
 rm -f containerd containerd-shim containerd-shim-runc-v2 ctr docker docker-init docker-proxy dockerd dockerd-rootless-setuptool.sh dockerd-rootless.sh rootlesskit rootlesskit-docker-proxy runc vpnkit
Usage
Daemon
With systemd (Highly recommended) Without systemd
The systemd unit file is installed as ~/.config/systemd/user/docker.service.

Use systemctl --user to manage the lifecycle of the daemon:


 systemctl --user start docker
To launch the daemon on system startup, enable the systemd service and lingering:


 systemctl --user enable docker
 sudo loginctl enable-linger $(whoami)
Starting Rootless Docker as a systemd-wide service (/etc/systemd/system/docker.service) is not supported, even with the User= directive.

Remarks about directory paths:

The socket path is set to $XDG_RUNTIME_DIR/docker.sock by default. $XDG_RUNTIME_DIR is typically set to /run/user/$UID.
The data dir is set to ~/.local/share/docker by default. The data dir should not be on NFS.
The daemon config dir is set to ~/.config/docker by default. This directory is different from ~/.docker that is used by the client.
Client
You need to specify either the socket path or the CLI context explicitly.

To specify the socket path using $DOCKER_HOST:


 export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
 docker run -d -p 8080:80 nginx
To specify the CLI context using docker context:


 docker context use rootless
rootless
Current context is now "rootless"
 docker run -d -p 8080:80 nginx
Best practices
Rootless Docker in Docker
To run Rootless Docker inside "rootful" Docker, use the docker:<version>-dind-rootless image instead of docker:<version>-dind.


 docker run -d --name dind-rootless --privileged docker:25.0-dind-rootless
The docker:<version>-dind-rootless image runs as a non-root user (UID 1000). However, --privileged is required for disabling seccomp, AppArmor, and mount masks.

Expose Docker API socket through TCP
To expose the Docker API socket through TCP, you need to launch dockerd-rootless.sh with DOCKERD_ROOTLESS_ROOTLESSKIT_FLAGS="-p 0.0.0.0:2376:2376/tcp".


 DOCKERD_ROOTLESS_ROOTLESSKIT_FLAGS="-p 0.0.0.0:2376:2376/tcp" \
  dockerd-rootless.sh \
  -H tcp://0.0.0.0:2376 \
  --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem
Expose Docker API socket through SSH
To expose the Docker API socket through SSH, you need to make sure $DOCKER_HOST is set on the remote host.


 ssh -l REMOTEUSER REMOTEHOST 'echo $DOCKER_HOST'
unix:///run/user/1001/docker.sock
 docker -H ssh://REMOTEUSER@REMOTEHOST run ...
Routing ping packets
On some distributions, ping does not work by default.

Add net.ipv4.ping_group_range = 0 2147483647 to /etc/sysctl.conf (or /etc/sysctl.d) and run sudo sysctl --system to allow using ping.

Exposing privileged ports
To expose privileged ports (< 1024), set CAP_NET_BIND_SERVICE on rootlesskit binary and restart the daemon.


 sudo setcap cap_net_bind_service=ep $(which rootlesskit)
 systemctl --user restart docker
Or add net.ipv4.ip_unprivileged_port_start=0 to /etc/sysctl.conf (or /etc/sysctl.d) and run sudo sysctl --system.

Limiting resources
Limiting resources with cgroup-related docker run flags such as --cpus, --memory, --pids-limit is supported only when running with cgroup v2 and systemd. See Changing cgroup version to enable cgroup v2.

If docker info shows none as Cgroup Driver, the conditions are not satisfied. When these conditions are not satisfied, rootless mode ignores the cgroup-related docker run flags. See Limiting resources without cgroup for workarounds.

If docker info shows systemd as Cgroup Driver, the conditions are satisfied. However, typically, only memory and pids controllers are delegated to non-root users by default.


 cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
memory pids
To allow delegation of all controllers, you need to change the systemd configuration as follows:


 mkdir -p /etc/systemd/system/user@.service.d
 cat > /etc/systemd/system/user@.service.d/delegate.conf << EOF
[Service]
Delegate=cpu cpuset io memory pids
EOF
 systemctl daemon-reload
Note

Delegating cpuset requires systemd 244 or later.

Limiting resources without cgroup
Even when cgroup is not available, you can still use the traditional ulimit and cpulimit, though they work in process-granularity rather than in container-granularity, and can be arbitrarily disabled by the container process.

For example:

To limit CPU usage to 0.5 cores (similar to docker run --cpus 0.5): docker run <IMAGE> cpulimit --limit=50 --include-children <COMMAND>

To limit max VSZ to 64MiB (similar to docker run --memory 64m): docker run <IMAGE> sh -c "ulimit -v 65536; <COMMAND>"

To limit max number of processes to 100 per namespaced UID 2000 (similar to docker run --pids-limit=100): docker run --user 2000 --ulimit nproc=100 <IMAGE> <COMMAND>

Troubleshooting
Unable to install with systemd when systemd is present on the system

 dockerd-rootless-setuptool.sh install
[INFO] systemd not detected, dockerd-rootless.sh needs to be started manually:
...
rootlesskit cannot detect systemd properly if you switch to your user via sudo su. For users which cannot be logged-in, you must use the machinectl command which is part of the systemd-container package. After installing systemd-container switch to myuser with the following command:


 sudo machinectl shell myuser@
Where myuser@ is your desired username and @ signifies this machine.

Errors when starting the Docker daemon
[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: operation not permitted

This error occurs mostly when the value of /proc/sys/kernel/unprivileged_userns_clone is set to 0:


 cat /proc/sys/kernel/unprivileged_userns_clone
0
To fix this issue, add kernel.unprivileged_userns_clone=1 to /etc/sysctl.conf (or /etc/sysctl.d) and run sudo sysctl --system.

[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: no space left on device

This error occurs mostly when the value of /proc/sys/user/max_user_namespaces is too small:


 cat /proc/sys/user/max_user_namespaces
0
To fix this issue, add user.max_user_namespaces=28633 to /etc/sysctl.conf (or /etc/sysctl.d) and run sudo sysctl --system.

[rootlesskit:parent] error: failed to setup UID/GID map: failed to compute uid/gid map: No subuid ranges found for user 1001 ("testuser")

This error occurs when /etc/subuid and /etc/subgid are not configured. See Prerequisites.

could not get XDG_RUNTIME_DIR

This error occurs when $XDG_RUNTIME_DIR is not set.

On a non-systemd host, you need to create a directory and then set the path:


 export XDG_RUNTIME_DIR=$HOME/.docker/xrd
 rm -rf $XDG_RUNTIME_DIR
 mkdir -p $XDG_RUNTIME_DIR
 dockerd-rootless.sh
Note

You must remove the directory every time you log out.

On a systemd host, log into the host using pam_systemd (see below). The value is automatically set to /run/user/$UID and cleaned up on every logout.

systemctl --user fails with "Failed to connect to bus: No such file or directory"

This error occurs mostly when you switch from the root user to a non-root user with sudo:


 sudo -iu testuser
 systemctl --user start docker
Failed to connect to bus: No such file or directory
Instead of sudo -iu <USERNAME>, you need to log in using pam_systemd. For example:

Log in through the graphic console
ssh <USERNAME>@localhost
machinectl shell <USERNAME>@
The daemon does not start up automatically

You need sudo loginctl enable-linger $(whoami) to enable the daemon to start up automatically. See Usage.

iptables failed: iptables -t nat -N DOCKER: Fatal: can't open lock file /run/xtables.lock: Permission denied

This error may happen with an older version of Docker when SELinux is enabled on the host.

The issue has been fixed in Docker 20.10.8. A known workaround for older version of Docker is to run the following commands to disable SELinux for iptables:


 sudo dnf install -y policycoreutils-python-utils && sudo semanage permissive -a iptables_t
docker pull errors
docker: failed to register layer: Error processing tar file(exit status 1): lchown <FILE>: invalid argument

This error occurs when the number of available entries in /etc/subuid or /etc/subgid is not sufficient. The number of entries required vary across images. However, 65,536 entries are sufficient for most images. See Prerequisites.

docker: failed to register layer: ApplyLayer exit status 1 stdout: stderr: lchown <FILE>: operation not permitted

This error occurs mostly when ~/.local/share/docker is located on NFS.

A workaround is to specify non-NFS data-root directory in ~/.config/docker/daemon.json as follows:


{"data-root":"/somewhere-out-of-nfs"}
docker run errors
docker: Error response from daemon: OCI runtime create failed: ...: read unix @->/run/systemd/private: read: connection reset by peer: unknown.

This error occurs on cgroup v2 hosts mostly when the dbus daemon is not running for the user.


 systemctl --user is-active dbus
inactive

 docker run hello-world
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: process_linux.go:385: applying cgroup configuration for process caused: error while starting unit "docker
-931c15729b5a968ce803784d04c7421f791d87e5ca1891f34387bb9f694c488e.scope" with properties [{Name:Description Value:"libcontainer container 931c15729b5a968ce803784d04c7421f791d87e5ca1891f34387bb9f694c488e"} {Name:Slice Value:"use
r.slice"} {Name:PIDs Value:@au [4529]} {Name:Delegate Value:true} {Name:MemoryAccounting Value:true} {Name:CPUAccounting Value:true} {Name:IOAccounting Value:true} {Name:TasksAccounting Value:true} {Name:DefaultDependencies Val
ue:false}]: read unix @->/run/systemd/private: read: connection reset by peer: unknown.
To fix the issue, run sudo apt-get install -y dbus-user-session or sudo dnf install -y dbus-daemon, and then relogin.

If the error still occurs, try running systemctl --user enable --now dbus (without sudo).

--cpus, --memory, and --pids-limit are ignored

This is an expected behavior on cgroup v1 mode. To use these flags, the host needs to be configured for enabling cgroup v2. For more information, see Limiting resources.

Networking errors
This section provides troubleshooting tips for networking in rootless mode.

Networking in rootless mode is supported via network and port drivers in RootlessKit. Network performance and characteristics depend on the combination of network and port driver you use. If you're experiencing unexpected behavior or performance related to networking, review the following table which shows the configurations supported by RootlessKit, and how they compare:

Network driver	Port driver	Net throughput	Port throughput	Source IP propagation	No SUID	Note
slirp4netns	builtin	Slow	Fast ✅	❌	✅	Default in a typical setup
vpnkit	builtin	Slow	Fast ✅	❌	✅	Default when slirp4netns isn't installed
slirp4netns	slirp4netns	Slow	Slow	✅	✅	
pasta	implicit	Slow	Fast ✅	✅	✅	Experimental; Needs pasta version 2023_12_04 or later
lxc-user-nic	builtin	Fast ✅	Fast ✅	❌	❌	Experimental
bypass4netns	bypass4netns	Fast ✅	Fast ✅	✅	✅	Note: Not integrated to RootlessKit as it needs a custom seccomp profile
For information about troubleshooting specific networking issues, see:

docker run -p fails with cannot expose privileged port
Ping doesn't work
IPAddress shown in docker inspect is unreachable
--net=host doesn't listen ports on the host network namespace
Network is slow
docker run -p does not propagate source IP addresses
docker run -p fails with cannot expose privileged port
docker run -p fails with this error when a privileged port (< 1024) is specified as the host port.


 docker run -p 80:80 nginx:alpine
docker: Error response from daemon: driver failed programming external connectivity on endpoint focused_swanson (9e2e139a9d8fc92b37c36edfa6214a6e986fa2028c0cc359812f685173fa6df7): Error starting userland proxy: error while calling PortManager.AddPort(): cannot expose privileged port 80, you might need to add "net.ipv4.ip_unprivileged_port_start=0" (currently 1024) to /etc/sysctl.conf, or set CAP_NET_BIND_SERVICE on rootlesskit binary, or choose a larger port number (>= 1024): listen tcp 0.0.0.0:80: bind: permission denied.
When you experience this error, consider using an unprivileged port instead. For example, 8080 instead of 80.


 docker run -p 8080:80 nginx:alpine
To allow exposing privileged ports, see Exposing privileged ports.

Ping doesn't work
Ping does not work when /proc/sys/net/ipv4/ping_group_range is set to 1 0:


 cat /proc/sys/net/ipv4/ping_group_range
1       0
For details, see Routing ping packets.

IPAddress shown in docker inspect is unreachable
This is an expected behavior, as the daemon is namespaced inside RootlessKit's network namespace. Use docker run -p instead.

--net=host doesn't listen ports on the host network namespace
This is an expected behavior, as the daemon is namespaced inside RootlessKit's network namespace. Use docker run -p instead.

Network is slow
Docker with rootless mode uses slirp4netns as the default network stack if slirp4netns v0.4.0 or later is installed. If slirp4netns is not installed, Docker falls back to VPNKit. Installing slirp4netns may improve the network throughput.

For more information about network drivers for RootlessKit, see RootlessKit documentation.

Also, changing MTU value may improve the throughput. The MTU value can be specified by creating ~/.config/systemd/user/docker.service.d/override.conf with the following content:


[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_MTU=INTEGER"
And then restart the daemon:


 systemctl --user daemon-reload
 systemctl --user restart docker
docker run -p does not propagate source IP addresses
This is because Docker in rootless mode uses RootlessKit's builtin port driver by default, which doesn't support source IP propagation. To enable source IP propagation, you can:

Use the slirp4netns RootlessKit port driver
Use the pasta RootlessKit network driver, with the implicit port driver
The pasta network driver is experimental, but provides improved throughput performance compared to the slirp4netns port driver. The pasta driver requires Docker Engine version 25.0 or later.

To change the RootlessKit networking configuration:

Create a file at ~/.config/systemd/user/docker.service.d/override.conf.

Add the following contents, depending on which configuration you would like to use:

slirp4netns


[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=slirp4netns"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
pasta network driver with implicit port driver


[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=pasta"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=implicit"
Restart the daemon:


 systemctl --user daemon-reload
 systemctl --user restart docker
For more information about networking options for RootlessKit, see:

Network drivers
Port drivers
Tips for debugging
Entering into dockerd namespaces

The dockerd-rootless.sh script executes dockerd in its own user, mount, and network namespaces.

For debugging, you can enter the namespaces by running nsenter -U --preserve-credentials -n -m -t $(cat $XDG_RUNTIME_DIR/docker.pid).