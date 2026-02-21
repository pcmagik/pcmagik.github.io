---
tags:
  - Docker
---

# Setting Up Docker with WSL

## 1. Install and Configure WSL

### Enable WSL Features
Open the command line as administrator and run the following commands to enable WSL features:

```pwersehell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
```pwersehell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### Install the Linux Kernel Update Package
Download and install the latest Linux kernel update package from Microsoft:

[Download wsl_update_x64.msi](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

After installation, restart Windows.

### Set WSL to Version 2
Run the following command as administrator to set WSL to version 2:

```bash
wsl --set-default-version 2
```

Restart Windows again to apply the changes.

---

## 2. Enable Systemd in WSL

### Check WSL Version
Ensure you are running WSL version `0.67.6` or higher. Check your version with:

```bash
wsl --version
```

If the command fails, upgrade to the Microsoft Store version of WSL. Run the following command to update:

```bash
wsl --update
```

## Alternatively, download the latest release from the:
[WSL release page](https://github.com/microsoft/WSL/releases).

### Configure Systemd
Edit the `wsl.conf` file to enable systemd. Open the file with sudo privileges:

```bash
sudo nano /etc/wsl.conf
```

Add the following lines:

```ini
[boot]
systemd=true
```

Save and exit the editor (`CTRL+O`, then `CTRL+X`).

### Restart WSL
Close your WSL instance and run the following command in PowerShell to restart WSL:

```bash
wsl.exe --shutdown
```

Launch your WSL instance again and verify systemd is running:

```bash
systemctl list-unit-files --type=service
```

For more details, refer to the [official blog post](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/).

---

## 3. Install Docker and Docker Compose v2 in WSL 2

### Install Docker
Run the following commands to install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Add Your User to the Docker Group
Add your user to the Docker group to avoid using `sudo` with Docker commands:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Verify Installation
Check that Docker and Docker Compose were installed successfully:

```bash
docker --version
docker compose version
```

---

## 4. Additional Step for Ubuntu 22.04 or Debian 10+
If you're using Ubuntu 22.04 or Debian 10+, configure `iptables` for compatibility:

```bash
sudo update-alternatives --config iptables
```

Select option `1` to use `iptables-legacy`.

### Start Docker
Start the Docker service:

```bash
sudo service docker start
```

Check the status of Docker:

```bash
sudo service docker status
```

> **Note:** You may need to reboot Windows or restart WSL after applying the `iptables` configuration to ensure networking inside your containers works correctly.
