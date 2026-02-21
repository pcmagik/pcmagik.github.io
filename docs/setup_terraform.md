# Terraform Setup Guide

A guide for installing Terraform and configuring it for infrastructure as code workflows.

---

## Installation

### Ubuntu / Debian

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt-get update && sudo apt-get install terraform
```

### RHEL / CentOS

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform
```

### Verify Installation

```bash
terraform --version
```

---

## Terraform Cloud Setup

### Generate API Token

1. Go to [Terraform Cloud](https://app.terraform.io/)
2. Navigate to **User Settings > Tokens > API Tokens**
3. Create a new token

### Configure Token on Server

```bash
mkdir -p ~/.terraform.d

cat > ~/.terraform.d/credentials.tfrc.json << 'EOF'
{
    "credentials": {
        "app.terraform.io": {
            "token": "your_api_token_here"
        }
    }
}
EOF

chmod 600 ~/.terraform.d/credentials.tfrc.json
```

---

## Workflow

### Initialize

```bash
terraform init
```

Downloads providers and modules, sets up the backend.

### Plan

```bash
terraform plan
```

Shows what changes will be made without applying them.

### Apply

```bash
terraform apply
```

Applies the planned changes. Add `-auto-approve` to skip confirmation (use with caution).

### Destroy

```bash
terraform destroy
```

Removes all resources managed by the configuration.

---

## Format and Validate

### Format HCL Files

```bash
# Format a single file
terraform fmt main.tf

# Format all files recursively
terraform fmt -recursive
```

### Validate Configuration

```bash
terraform validate
```

> **Tip:** Always run `fmt` and `validate` before `plan` to catch syntax issues early.

### Batch Format and Validate (Packer Example)

```bash
#!/bin/bash
# Format all Packer HCL files
for dir in */; do
  if [ -f "${dir}packer.pkr.hcl" ]; then
    packer fmt "${dir}packer.pkr.hcl"
    packer validate -var-file variables.pkrvars.hcl "${dir}packer.pkr.hcl"
  fi
done
```

---

## Proxmox Provider (Example)

### Provider Configuration

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "~> 3.0"
    }
  }
}

provider "proxmox" {
  pm_api_url          = "https://proxmox.example.com:8006/api2/json"
  pm_api_token_id     = "your_username@pam!terraform"
  pm_api_token_secret = "your_api_token_here"
  pm_tls_insecure     = true
}
```

> **Warning:** Never hardcode credentials. Use environment variables instead:

```bash
export PM_API_TOKEN_ID="your_username@pam!terraform"
export PM_API_TOKEN_SECRET="your_api_token_here"
```

### VM Resource Example

```hcl
resource "proxmox_vm_qemu" "vm" {
  name        = "terraform-vm"
  target_node = "pve"
  clone       = "ubuntu-template"
  cores       = 2
  memory      = 2048

  disk {
    storage = "local-lvm"
    size    = "20G"
    type    = "scsi"
  }

  network {
    model  = "virtio"
    bridge = "vmbr0"
  }
}
```

---

## Best Practices

- Store state remotely (Terraform Cloud, S3, Consul)
- Use `.tfvars` files for environment-specific values
- Never commit credentials or `.tfstate` files to git
- Add to `.gitignore`:
```text
*.tfstate
*.tfstate.backup
.terraform/
*.tfvars
```
