---
tags:
  - DevOps
  - Ansible
---

# Ansible Setup Guide

A guide for installing Ansible and getting started with infrastructure automation.

---

## Installation

### Ubuntu

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### RHEL / AlmaLinux / CentOS

```bash
sudo dnf update -y
sudo dnf install -y epel-release
sudo dnf install ansible -y
```

### Verify Installation

```bash
ansible --version
```

---

## Inventory Configuration

The inventory file defines the hosts Ansible manages.

### Default Location

```
/etc/ansible/hosts
```

### Example Inventory

```ini
[webservers]
server1 ansible_host=10.0.0.11
server2 ansible_host=10.0.0.12
server3 ansible_host=10.0.0.13

[dbservers]
db1 ansible_host=10.0.0.21

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=your_username
ansible_ssh_private_key_file=~/.ssh/my-server-key
```

### Verify Inventory

```bash
ansible-inventory --list -y
```

---

## Ad-Hoc Commands

Quick commands without writing a playbook.

### Ping All Hosts

```bash
ansible all -m ping
```

### Run a Shell Command

```bash
ansible all -a "df -h"
ansible all -a "uptime"
```

### Install a Package

```bash
# On Debian/Ubuntu hosts
ansible webservers -m apt -a "name=vim state=latest" --become

# On RHEL/CentOS hosts
ansible dbservers -m dnf -a "name=vim state=latest" --become
```

### Target Specific Hosts or Groups

```bash
# Target a group
ansible webservers -m ping

# Target specific hosts
ansible server1:server2 -m ping
```

---

## Playbook Basics

Playbooks are YAML files defining automation tasks.

### Example: Install Docker

```yaml
# install-docker.yml
---
- name: Install Docker on Ubuntu servers
  hosts: webservers
  become: true

  tasks:
    - name: Install prerequisites
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present
        update_cache: true

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: "deb https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker
      apt:
        name: docker-ce
        state: present
        update_cache: true

    - name: Start and enable Docker
      systemd:
        name: docker
        state: started
        enabled: true

    - name: Add user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: true
```

### Run a Playbook

```bash
ansible-playbook install-docker.yml
```

### Dry Run (Check Mode)

```bash
ansible-playbook install-docker.yml --check
```

### Verbose Output

```bash
ansible-playbook install-docker.yml -v    # basic
ansible-playbook install-docker.yml -vvv  # detailed
```

---

## Ansible + Docker Integration

### Run Containers via Ansible

```yaml
- name: Run Nginx container
  community.docker.docker_container:
    name: nginx
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    restart_policy: unless-stopped
```

> **Note:** Requires the `community.docker` collection:
```bash
ansible-galaxy collection install community.docker
```

---

## Useful Commands

```bash
# List all hosts
ansible all --list-hosts

# Test connectivity
ansible all -m ping

# Gather facts about hosts
ansible all -m setup

# Run playbook with specific inventory
ansible-playbook -i inventory.ini playbook.yml

# Limit to specific hosts
ansible-playbook playbook.yml --limit server1
```
