# Local LLM Setup Guide

A guide for running large language models locally using NVIDIA GPU passthrough, Ollama, and Open WebUI.

---

## NVIDIA Driver Installation (Ubuntu)

### Add Driver Repository

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

### Install Driver

```bash
# Install specific version
sudo apt install nvidia-driver-570

# Or use auto-detection
sudo ubuntu-drivers autoinstall
```

### Reboot and Verify

```bash
sudo reboot
```

After reboot:

```bash
# Check GPU status
nvidia-smi

# Load NVIDIA module
sudo modprobe nvidia

# Detailed GPU info
lspci -v | grep -A 12 NVIDIA
```

---

## GPU Passthrough Verification (Proxmox VMs)

### Check IOMMU Status

```bash
dmesg | grep -i iommu
```

### Check VFIO Configuration

```bash
cat /etc/default/grub | grep iommu
```

### SecureBoot Status

```bash
mokutil --sb-state
```

> **Note:** If SecureBoot is enabled, you may need to disable it for NVIDIA drivers to load:
```bash
sudo apt install mokutil
sudo mokutil --disable-validation
```

### Driver Verification

```bash
modinfo nvidia | grep version
dkms status
```

### Monitor GPU in Real-Time

```bash
watch -n 0.5 nvidia-smi
```

---

## Ollama Installation

### Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Basic Usage

```bash
# List installed models
ollama list

# Run a model
ollama run llama3.1:8b

# Stop a running model
ollama stop llama3.1:8b

# Pull a model without running
ollama pull deepseek-r1:14b
```

---

## Open WebUI (Docker)

Open WebUI provides a ChatGPT-like interface for Ollama models.

### Default Setup (Ollama on Same Machine)

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Remote Ollama Server

```bash
docker run -d -p 3000:8080 \
  -e OLLAMA_BASE_URL=https://ollama.example.com \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### With NVIDIA GPU Support

```bash
docker run -d -p 3000:8080 \
  --gpus all \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:cuda
```

### OpenAI API Only

```bash
docker run -d -p 3000:8080 \
  -e OPENAI_API_KEY=your_api_token_here \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Bundled (Ollama + WebUI in One Container)

**With GPU:**
```bash
docker run -d -p 3000:8080 \
  --gpus=all \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

**CPU Only:**
```bash
docker run -d -p 3000:8080 \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

---

## Access

Open your browser and navigate to:

```
http://server.example.com:3000
```

Create an admin account on first login.
