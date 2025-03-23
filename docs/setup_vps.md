# Setup new VPS with Ubuntu 24.04 LTS Minimal ARM64

# Set the timezone to Warsaw
sudo timedatectl set-timezone Europe/Warsaw

# Minimize the installation
sudo unminimize

# Install git and GitHub CLI
sudo apt-get update
sudo apt-get install git gh -y

# Add a new user and grant sudo privileges
# Replace 'pcmagik' with your desired username
sudo adduser pcmagik 
sudo usermod -aG sudo pcmagik

# Set up SSH keys for the new user
## 1. Create the `.ssh` Directory
```bash
mkdir -p ~/.ssh
cd ~/.ssh
```

## 2. Generate an SSH Key Pair
```bash
ssh-keygen -b 4096 -t rsa -f pcmagik-zurich-arm-docker-pcmagik-com
```

## 3. Add the Public Key to `authorized_keys`
You can use one of the following methods:

### Method 1: Append the Key
```bash
cat ~/.ssh/pcmagik-zurich-arm-docker-pcmagik-com.pub >> ~/.ssh/authorized_keys
```
## 4. Set Permissions
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

## 5. Install `putty-tools`
```bash
sudo apt install putty-tools
```

## 6. Convert the Private Key to PPK Format
```bash
puttygen ~/pcmagik-zurich-arm-docker-pcmagik-com -o ~/.ssh/pcmagik-zurich-arm-docker-gronioss-pamagik-com.ppk
```

## 7. Copy Keys to the `ubuntu` User's Home Directory
```bash
sudo cp pcmagik-zurich-arm-docker-pcmagik-com /home/ubuntu/
sudo cp pcmagik-zurich-arm-docker-pcmagik-com.pub /home/ubuntu/
```

## 8. Change Ownership of the Keys
```bash
sudo chown ubuntu:ubuntu /home/ubuntu/pcmagik-zurich-arm-docker-pcmagik-com
sudo chown ubuntu:ubuntu /home/ubuntu/pcmagik-zurich-arm-docker-pcmagik-com.pub
```





# GitHub Authentication
gh auth login


# Set global git configurations
git config --global user.name "Mateusz Piekut"
git config --global user.email "serwis.pcmagik@gmail.com"

# Update, upgrade and clean the system
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo apt-get autoremove -y

# Install comprehensive tools for network management, system monitoring, and more
sudo apt update && sudo apt install mc nano net-tools iputils-ping curl wget git htop tcpdump traceroute vim zip unzip neofetch ncat cifs-utils bash-completion hstr -y

# Install CrowdSec for system protection
curl -s https://install.crowdsec.net | sudo sh

sudo apt install crowdsec

sudo apt install crowdsec-firewall-bouncer-iptables

sudo systemctl restart crowdsec



# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to Docker group and refresh group membership
sudo usermod -aG docker $USER
newgrp docker

# Test Docker installation
sudo docker run hello-world

# Run Portainer
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

