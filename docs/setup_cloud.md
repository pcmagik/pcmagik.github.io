---
tags:
  - Cloud
---

# Cloud-Init Setup Guide

A guide for using cloud-init to automate virtual machine provisioning, with focus on Proxmox templates.

---

## What is Cloud-Init?

Cloud-init is the industry standard for early-stage initialization of virtual machines. It handles:

- Setting hostname and timezone
- Creating users and SSH keys
- Installing packages
- Running custom scripts
- Network configuration

---

## Cloud-Init Config for Proxmox VMs

### User Data Example

```yaml
#cloud-config

hostname: server.example.com
timezone: Europe/Warsaw

users:
  - name: your_username
    groups: sudo, docker
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-rsa AAAA...your_public_key_here

package_update: true
package_upgrade: true
packages:
  - curl
  - wget
  - git
  - htop
  - vim
  - net-tools

runcmd:
  - systemctl enable --now qemu-guest-agent
```

---

## Cloud-Init with Tailscale

Automatically install and configure Tailscale on first boot:

```yaml
#cloud-config

runcmd:
  # Install Tailscale
  - ['sh', '-c', 'curl -fsSL https://tailscale.com/install.sh | sh']

  # Enable IP forwarding for exit node
  - ['sh', '-c', "echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf && echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf && sudo sysctl -p /etc/sysctl.d/99-tailscale.conf"]

  # Authenticate with auth key (generate at https://login.tailscale.com/admin/settings/keys)
  - ['tailscale', 'up', '--authkey=your_api_token_here']

  # Optional: Enable Tailscale SSH
  - ['tailscale', 'set', '--ssh']

  # Optional: Configure as exit node
  - ['tailscale', 'set', '--advertise-exit-node']
```

---

## Proxmox Cloud-Init Templates

### Download Cloud Images

Common cloud image sources:

| Distribution | Image URL |
|---|---|
| Ubuntu 24.04 | `https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img` |
| Ubuntu 22.04 | `https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img` |
| Debian 12 | `https://cdimage.debian.org/cdimage/cloud/bookworm/latest/debian-12-generic-amd64.qcow2` |
| AlmaLinux 9 | `https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/AlmaLinux-9-GenericCloud-latest.x86_64.qcow2` |
| Fedora 38 | `https://download.fedoraproject.org/pub/fedora/linux/releases/38/Cloud/x86_64/images/Fedora-Cloud-Base-38-1.6.x86_64.qcow2` |

### Create a Template

```bash
# Download image
wget -O /tmp/ubuntu-24.04.img \
  https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

# Install tools for image customization
apt install libguestfs-tools -y

# Customize image (install qemu-guest-agent)
virt-customize -a /tmp/ubuntu-24.04.img \
  --install qemu-guest-agent \
  --run-command 'systemctl enable qemu-guest-agent'

# Create VM
qm create 9100 --name ubuntu-24.04-template \
  --memory 2048 --cores 2 \
  --net0 virtio,bridge=vmbr0

# Import disk
qm importdisk 9100 /tmp/ubuntu-24.04.img local-lvm

# Attach disk
qm set 9100 --scsihw virtio-scsi-pci \
  --scsi0 local-lvm:vm-9100-disk-0

# Add cloud-init drive
qm set 9100 --ide2 local-lvm:cloudinit

# Set boot order
qm set 9100 --boot c --bootdisk scsi0

# Add serial console
qm set 9100 --serial0 socket --vga serial0

# Resize disk
qm resize 9100 scsi0 +18G

# Convert to template
qm template 9100
```

### Clone from Template

```bash
# Full clone
qm clone 9100 200 --name my-new-vm --full

# Start the VM
qm start 200
```

---

## Network Configuration via Cloud-Init

### Proxmox GUI

In Proxmox web interface:
1. Select VM > **Cloud-Init** tab
2. Set: User, Password, SSH key, IP config
3. **Regenerate Image** after changes

### CLI

```bash
# Set cloud-init user and SSH key
qm set 200 --ciuser your_username
qm set 200 --sshkeys ~/.ssh/my-server-key.pub

# Set static IP
qm set 200 --ipconfig0 ip=10.0.0.50/24,gw=10.0.0.1

# Set DNS
qm set 200 --nameserver 1.1.1.1
qm set 200 --searchdomain example.com
```

---

## Free-Tier Options for Learning

Several cloud providers offer free tiers suitable for learning cloud-init:

- **Oracle Cloud** - Always Free (ARM instances, 24GB RAM, 4 OCPU)
- **Google Cloud** - Free Tier (e2-micro instance)
- **AWS** - Free Tier (t2.micro for 12 months)
- **Azure** - Free Tier (B1s for 12 months)

All support cloud-init for VM initialization.
