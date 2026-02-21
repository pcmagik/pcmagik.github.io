---
tags:
  - Proxmox
---

# Proxmox VE Setup Guide

A comprehensive guide for Proxmox Virtual Environment configuration, optimization, and management.

---

## Post-Install Setup

### Remove Subscription Notice and Update Repositories

Run the community post-install script or configure manually:

```bash
# Add no-subscription repository
echo "deb http://download.proxmox.com/debian/pve $(grep 'VERSION_CODENAME' /etc/os-release | cut -d= -f2) pve-no-subscription" \
  | tee /etc/apt/sources.list.d/pve-no-subscription.list

# Disable enterprise repository
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Update system
apt-get update && apt-get dist-upgrade -y
```

---

## Network Configuration

### Edit Network Interfaces

```bash
nano /etc/network/interfaces
```

### Bridge Configuration (Standard)

```text
auto vmbr0
iface vmbr0 inet static
    address 10.0.0.100/24
    gateway 10.0.0.1
    bridge_ports eth0
    bridge_stp off
    bridge_fd 0
```

### VLAN Configuration

```text
auto vmbr1
iface vmbr1 inet static
    address 10.0.1.100/24
    gateway 10.0.1.1
    bridge_ports eth0.10
    bridge_stp off
    bridge_fd 0
    vlan-raw-device eth0
```

### Apply Network Changes

```bash
# Standard restart
systemctl restart networking

# Without full restart
ifreload -a

# Restart a specific bridge
ifdown vmbr0 && ifup vmbr0
```

> **Warning:** Changing IP remotely via SSH may disconnect you. Ensure you have console access (IPMI, physical) before modifying network settings.

### Verify Configuration

```bash
ip addr show
ping -c 4 10.0.0.1
```

---

## ZFS Optimization for SSD

### Disable Unnecessary HA Services (Single Node)

```bash
systemctl disable pve-ha-lrm.service
systemctl disable pve-ha-crm.service
```

### Store Logs in RAM

Edit `/etc/systemd/journald.conf`:

```ini
[Journal]
Storage=volatile
```

```bash
systemctl restart systemd-journald
```

### Enable Autotrim

```bash
zpool set autotrim=on rpool
```

### Disable atime

```bash
zfs set atime=off rpool
```

### Use Raw Disk Images

```bash
qm set <VMID> --scsihw virtio-scsi-pci --virtio0 /dev/zvol/rpool/vm-<VMID>-disk-0
```

### Keep Pool Below 80% Capacity

ZFS performance degrades significantly above 80% usage. Monitor regularly:

```bash
zpool list
```

### ZRAM Instead of Swap on SSD

```bash
apt install zram-tools -y
systemctl enable --now zramswap.service

# Verify ZRAM is active
swapon --show
```

### Monitoring

```bash
# Check SSD wear
smartctl -a /dev/sda

# Real-time ZFS I/O
watch -n 1 zpool iostat rpool
```

---

## Firewall Configuration

### Enable Firewall

```bash
pvecm set --firewall 1
```

### Add Rules via CLI

```bash
# Allow SSH on VM 101
pvesh set /cluster/firewall/rules \
  -type in -action accept -dport 22

# View rules
pvesh get /nodes/<node_name>/firewall/rules
```

### Firewall Config File

```bash
nano /etc/pve/firewall/cluster.fw
```

---

## Disk Passthrough (HDD/SSD to VM)

### Identify Disks by ID

```bash
ls -l /dev/disk/by-id/ | grep -v part
```

### Attach Disk to VM

```bash
# Attach disk by-id to VM (recommended over /dev/sdX)
qm set <VMID> -scsi1 /dev/disk/by-id/ata-VENDOR_MODEL_SERIAL
qm set <VMID> -scsi2 /dev/disk/by-id/ata-VENDOR_MODEL_SERIAL
```

> **Important:** Always use `/dev/disk/by-id/` paths instead of `/dev/sdX` as device letters can change after reboot.

### List Hardware

```bash
lshw -class disk -class storage
lshw -C disk
```

---

## GPU Passthrough

### GRUB Configuration for IOMMU

Edit `/etc/default/grub`:

**Intel CPU:**
```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

**AMD CPU:**
```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```

```bash
update-grub
reboot
```

### Verify IOMMU

```bash
dmesg | grep -i iommu
```

---

## Ceph Storage Cluster

> **Note:** Only configure Ceph after establishing a reliable network mesh between nodes.

### Install Ceph (All Nodes)

```bash
pveceph install --repository no-subscription
```

### Initialize Ceph (First Node)

```bash
pveceph init --network 10.0.0.0/24
```

### Create Monitors

```bash
# Node 1
pveceph mon create --mon-address 10.0.0.1

# Node 2
pveceph mon create --mon-address 10.0.0.2

# Node 3
pveceph mon create --mon-address 10.0.0.3
```

### Add Managers

Via GUI: `Datacenter > Node > Ceph > Monitor > Create Manager`

### Add OSDs

Via GUI: `Datacenter > Node > Ceph > OSD > Create OSD`

> **Tip:** If no disks are available, wipe old filesystems via `Datacenter > Node > Disks`.

### Create Pool

Via GUI: `Datacenter > Node > Ceph > Pools > Create`

Example pool name: `vm-disks`

### Configure HA

1. Set `Cluster Resource Scheduling` to `ha-rebalance-on-start=1`
2. Set `HA Settings > shutdown_policy` to `migrate`
3. Leave migration settings as default

---

## Batch VM Management

### Delete Multiple VMs in a Loop

```bash
for i in $(seq 9001 9050); do
    if qm list | awk '{print $1}' | grep -q "^$i$"; then
        echo "Deleting VM $i"
        qm destroy $i --purge --skiplock
    fi
done
```

### Preview Only (Dry Run)

```bash
for i in $(seq 9001 9050); do
    if qm list | awk '{print $1}' | grep -q "^$i$"; then
        echo "Would delete VM $i"
    fi
done
```

---

## Upgrade Proxmox (8.x to 9.x)

### Prerequisites

- Proxmox VE 8.4.1 or higher
- At least 10GB free disk space
- Root access
- Internet connectivity

### Upgrade Steps

1. **Update current installation:**
```bash
apt update && apt dist-upgrade -y
```

2. **Run pre-upgrade checks:**
```bash
pve8to9 --full
```

3. **Update repositories for PVE 9:**
```bash
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/*.list
```

4. **Perform the upgrade:**
```bash
apt update && apt dist-upgrade -y
```

5. **Reboot:**
```bash
systemctl reboot
```

> **Warning:** Always create backups before upgrading. Test on a non-production node first. Known issues include GRUB UEFI changes and network interface naming changes.
