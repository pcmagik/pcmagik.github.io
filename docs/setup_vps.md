#!/bin/bash

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

