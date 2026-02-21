---
tags:
  - DevOps
  - Packer
---

# Packer Setup Guide

A guide for installing HashiCorp Packer and creating automated machine images for various platforms.

---

## What is Packer?

Packer is a tool for building identical machine images for multiple platforms from a single source configuration. It automates the creation of VM templates, cloud images, and container images.

---

## Installation

### Ubuntu / Debian

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt-get update && sudo apt-get install packer
```

### RHEL / CentOS

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install packer
```

### Verify Installation

```bash
packer --version
```

---

## Format and Validate

### Format HCL Files

```bash
packer fmt template.pkr.hcl
```

### Validate Configuration

```bash
packer validate template.pkr.hcl

# With variable file
packer validate -var-file variables.pkrvars.hcl template.pkr.hcl
```

> **Tip:** Always run `fmt` and `validate` before building.

---

## Boot Commands Reference

Boot commands simulate keyboard input during OS installation. They vary by distribution and version.

### Ubuntu 24.04 (Autoinstall)

```hcl
boot_command = [
  "<wait3s>c<wait3s>",
  "linux /casper/vmlinuz --- autoinstall ds=nocloud-net;s=http://{{ .HTTPIP }}:{{ .HTTPPort }}/",
  "<enter><wait>",
  "initrd /casper/initrd",
  "<enter><wait>",
  "boot",
  "<enter>"
]
```

### Ubuntu 22.04 (Autoinstall)

```hcl
boot_command = [
  "<esc><wait>",
  "e<wait>",
  "<down><down><down><end>",
  "<bs><bs><bs><bs><wait>",
  "autoinstall ds=nocloud-net\\;s=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ ---<wait>",
  "<f10><wait>"
]
boot      = "c"
boot_wait = "5s"
```

### Ubuntu 20.04 (Autoinstall)

```hcl
boot_command = [
  "<esc><wait><esc><wait>",
  "<f6><wait><esc><wait>",
  "<bs><bs><bs><bs><bs>",
  "autoinstall ds=nocloud-net;s=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ ",
  "--- <enter>"
]
boot      = "c"
boot_wait = "5s"
```

### Debian 12 (Preseed)

```hcl
boot_command = [
  "<esc><wait>",
  "auto console-setup/ask_detect=false ",
  "debconf/frontend=noninteractive fb=false ",
  "url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg",
  "<enter>"
]
```

### Debian 10 (Preseed)

```hcl
boot_command = [
  "<esc><wait>",
  "auto install ",
  "console-keymaps-at/keymap=us console-setup/ask_detect=false ",
  "debconf/frontend=noninteractive ",
  "url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg",
  "<enter><wait>"
]
```

---

## Proxmox Builder (Overview)

### Example Template

```hcl
packer {
  required_plugins {
    proxmox = {
      version = ">= 1.1.0"
      source  = "github.com/hashicorp/proxmox"
    }
  }
}

source "proxmox-iso" "ubuntu" {
  proxmox_url              = "https://proxmox.example.com:8006/api2/json"
  username                 = "your_username@pam!packer"
  token                    = "your_api_token_here"
  insecure_skip_tls_verify = true

  node                 = "pve"
  vm_name              = "ubuntu-template"
  template_description = "Ubuntu 24.04 template built by Packer"

  iso_file    = "local:iso/ubuntu-24.04-live-server-amd64.iso"
  unmount_iso = true

  cores  = 2
  memory = 2048

  disks {
    storage_pool = "local-lvm"
    disk_size    = "20G"
    type         = "scsi"
  }

  network_adapters {
    bridge = "vmbr0"
    model  = "virtio"
  }

  http_directory = "http"
  ssh_username   = "your_username"
  ssh_password   = "your_password_here"
  ssh_timeout    = "20m"
}

build {
  sources = ["source.proxmox-iso.ubuntu"]

  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get upgrade -y",
      "sudo apt-get install -y qemu-guest-agent",
      "sudo systemctl enable qemu-guest-agent"
    ]
  }
}
```

---

## Variable Files and Credentials

### variables.pkrvars.hcl

```hcl
proxmox_url      = "https://proxmox.example.com:8006/api2/json"
proxmox_username = "your_username@pam!packer"
proxmox_token    = "your_api_token_here"
ssh_username     = "your_username"
ssh_password     = "your_password_here"
```

> **Warning:** Never commit variable files with credentials to git. Add to `.gitignore`:
```text
*.pkrvars.hcl
*.auto.pkrvars.hcl
```

### Use Environment Variables Instead

```bash
export PKR_VAR_proxmox_token="your_api_token_here"
export PKR_VAR_ssh_password="your_password_here"
```

---

## Build Workflow

```bash
# Initialize plugins
packer init template.pkr.hcl

# Format
packer fmt template.pkr.hcl

# Validate
packer validate -var-file variables.pkrvars.hcl template.pkr.hcl

# Build
packer build -var-file variables.pkrvars.hcl template.pkr.hcl
```
