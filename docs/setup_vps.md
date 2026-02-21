# Complete Configuration Guide for Ubuntu 24.04 LTS ARM64 VPS Server

## Set the Timezone to Warsaw
```bash
sudo timedatectl set-timezone Europe/Warsaw
```

## Minimize the Installation
```bash
sudo unminimize
```

## Install Git and GitHub CLI
```bash
sudo apt-get update
sudo apt-get install git gh -y
```

## Add a New User and Grant Sudo Privileges
Replace `your_username` with your desired username.
```bash
sudo adduser your_username
sudo usermod -aG sudo your_username
```

## Add no password sudo access for the new user
```bash
echo "your_username ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/your_username
sudo chmod 0440 /etc/sudoers.d/your_username
```

## Set Up SSH Keys for the New User

### 1. Create the `.ssh` Directory
```bash
mkdir -p ~/.ssh
cd ~/.ssh
```

### 2. Generate an SSH Key Pair
```bash
ssh-keygen -b 4096 -t rsa -f my-server-key
```

### 3. Add the Public Key to `authorized_keys`
#### Method 1: Append the Key
```bash
cat ~/.ssh/my-server-key.pub >> ~/.ssh/authorized_keys
```

### 4. Set Permissions
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

### 5. Install `putty-tools`
```bash
sudo apt install putty-tools
```

### 6. Convert the Private Key to PPK Format

```bash
puttygen ~/my-server-key -o ~/.ssh/my-server-key.ppk
```
---
## Chicken and Egg Problem with SSH Keys
Now we need to copy the keys to local machine, and we have three options:
### Option 1: Use `scp` (if password authentication is enabled)
#### For Windows (PowerShell):
```powershell
scp your_user@server_address:~/.ssh/my-server-key C:\Users\YourUser\Downloads\
scp your_user@server_address:~/.ssh/my-server-key.pub C:\Users\YourUser\Downloads\
scp your_user@server_address:~/.ssh/my-server-key.ppk C:\Users\YourUser\Downloads\
```

#### For Linux/macOS:
```bash
scp your_user@server_address:~/.ssh/my-server-key ~/Downloads/
scp your_user@server_address:~/.ssh/my-server-key.pub ~/Downloads/
scp your_user@server_address:~/.ssh/my-server-key.ppk ~/Downloads/
```

#### Alternative: Use `ssh-copy-id`
If you have password authentication to the server, you can use `ssh-copy-id` to copy the public key to the server. This will allow you to log in without a password. But you need to generate the keys first on local machine and copy them to the server.
```bash
ssh-copy-id -i ~/.ssh/my-server-key.pub your_user@server_address
```

### Option 2: Copy via another user (if you only have SSH key authentication)
#### 2.1 Copy Keys to the `ubuntu` User's Home Directory
```bash
sudo cp my-server-key /home/ubuntu/
sudo cp my-server-key.pub /home/ubuntu/
```

#### 2.2 Change Ownership and Permissions of the Keys
```bash
sudo chown ubuntu:ubuntu /home/ubuntu/my-server-key
sudo chown ubuntu:ubuntu /home/ubuntu/my-server-key.pub
sudo chmod 600 /home/ubuntu/my-server-key
sudo chmod 644 /home/ubuntu/my-server-key.pub
```

### Option 3: Manual Copy-Paste
Just use "cat" command to show private and public keys, copy and save it to your local machine:
```bash
cat ~/.ssh/my-server-key.pub
cat ~/.ssh/my-server-key
```

## BONUS
If You want to block root login and password authentication, you can do it with the following command:
```bash
sudo sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config && sudo sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && sudo systemctl restart sshd
```
If you want to allow password authentication for the new user, you can do it with the following command:
```bash
sudo sed -i 's/^PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config && sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config && sudo systemctl restart sshd
```
When You have password authentication enabled, you can use the following command to connect to the server, make directory and copy the public key to the server:
```bash
ssh użytkownik@adres_serwera -p port "mkdir -p ~/.ssh && chmod 700 ~/.ssh" && scp -P port ~/.ssh/lokalizacja/klucz.pub użytkownik@adres_serwera:~/.ssh/authorized_keys
```

---

## GitHub Authentication
```bash
gh auth login
```

## Set Global Git Configurations
```bash
git config --global user.name "Your Name"
git config --global user.email "user@example.com"
```

## Update, Upgrade, and Clean the System
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo apt-get autoremove -y
```

## Install Comprehensive Tools
```bash
sudo apt update && sudo apt install mc nano net-tools iputils-ping curl wget git htop tcpdump traceroute vim zip unzip neofetch ncat cifs-utils bash-completion hstr -y
```

## Install CrowdSec for System Protection
```bash
curl -s https://install.crowdsec.net | sudo sh
sudo apt install crowdsec
sudo apt install crowdsec-firewall-bouncer-iptables
sudo systemctl restart crowdsec
```

## Install Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Add Current User to Docker Group and Refresh Group Membership
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Test Docker Installation
```bash
sudo docker run hello-world
```

### Run Portainer
```bash
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```