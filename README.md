# Deploying a Linux Server (Ubuntu 24.04) on Proxmox VE

This guide explains how to create and configure an Ubuntu Server virtual machine on Proxmox VE. The VM will be used to host Immich and can also serve as a foundation for other self-hosted applications.

In addition to installing Ubuntu Server, this guide covers Docker installation, initial SSH access, and Tailscale VPN setup.

---
# Immich Installation Guide

## Table of Contents

- [Create a New VM](#create-a-new-vm)
- [Ubuntu Server Installation](#ubuntu-server-installation)
- [Post-Installation Setup](#post-installation-setup)
    - [Docker](#install-docker)
    - [SSH](#ssh)
    - [Tailscale](#tailscale)
---

## 1. Download Ubuntu Server ISO

First, download the Ubuntu Server image:

* Go to the official Ubuntu website: https://ubuntu.com/download/server
* Download the latest **LTS version (recommended)** I installed Ubuntu 24.04.4-live-server-amd64

After downloading, upload the ISO to Proxmox:

```text id="iso_upload"
PVE → local (storage) → ISO Images → Upload
```
![Uploading ISO](https://github.com/MikeMilenk/Immich-deployment/blob/ff3d949471ce37fbe5e0a1647dea83eca7c39248/images/Download%20ISO%20Images.png)

---

## Create a New VM

In Proxmox web interface:

```text id="vm_create"
Click "Create VM"
```
![Create VM](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Create%20VM.png)

### Basic Settings:

* **Node:** select your Proxmox host (`pve` in my case)
* **VM ID:** auto or custom (111 in my case)
* **Name:** I named as `Immich`

Click **Next**
![General](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/General.png)

---

## OS Setup

* Storage **local**
* Select **Use CD/DVD disc image file (ISO)**
* Choose the uploaded **Ubuntu Server ISO**
* Guest OS type:
  * Type: **Linux**
  * Version: **Ubuntu (latest)**

Click **Next**
![OS](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/OS.png)

---

## System Configuration

Recommended settings:

* **Display:** Default
* **Machine:** q35
* **BIOS:** OVMF (UEFI) *(recommended for modern setups)*
* Enable **QEMU Agent** (can be installed later in OS)

Click **Next**
![System](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/System.png)

---

## Disk Configuration

Here you select where the VM disk will be stored.

* **Storage:** select your ZFS pool (`immich` in my case)
* **Disk size:** I allocated 150 GB (should be more than enough for running Ubuntu server only)
* **Bus/Device:** VirtIO SCSI (recommended)

Click **Next**
![Disks](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Disks.png)

---

## CPU and Memory

* **CPU:** 2–4 cores (depends on workload)
* **Memory:** 8 GB minimum for Ubuntu Server with Immich installed. 16 GB is better. Note: 7812 MiB is ~8192 MB or 8 GB

Click **Next**
![CPU](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/CPU.png)
![Memory](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Memory.png)

---

## Network Configuration (leave defaults)

* **Bridge:** `vmbr0`
* Model: **VirtIO (paravirtualized)**

Click **Next**
![Network](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Network.png)

---

## Finish Setup

Review settings and click **Finish**.
![Confirmation](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Confirmation.png)

---

## 2. Ubuntu Server Installation

- Update to new installer
- Keyboard layout: English (US)
- Type: Ubuntu Server (only)
![Installation type](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/c660d90bde94a8779a8d71cbc55400a94cbb6c03/Images/Type%20of%20install.png)

# Network Configuration (leave defaults)
- Proxy: empty (default)
- Mirror: auto-detect → Done
![Mirror configs](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/c660d90bde94a8779a8d71cbc55400a94cbb6c03/Images/Mirror%20Address.png)

You will get a new IP address and DHCP configuration here. You can change it later.
![Network Config](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/1417b632c43376f130649b70bdf62d50bb5d6ad8/Images/Networks%20Config.png)

# Storage
- Use entire disk
- Enable LVM group
![Storage](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/1417b632c43376f130649b70bdf62d50bb5d6ad8/Images/Storage.png)

# User setup
- Create username and password

# Ubuntu Pro
- Skip

# SSH
- Install OpenSSH server (enable)
- I did not import SSH keys, since I don't hae them. Will create a new SSH connection
![SSH Configs](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/1417b632c43376f130649b70bdf62d50bb5d6ad8/Images/SSH%20Config.png)

# Snaps
- Leave default
![Snaps features](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/1417b632c43376f130649b70bdf62d50bb5d6ad8/Images/Server%20snaps.png)

# Finish
- Install Ubuntu

---

## Post-Installation (Important)

After installation, remove ISO:

```text id="iso_remove"
VM → Hardware → CD/DVD Drive → Do not use any media
```
![Remove CD-DVD](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/9cee3956199e0f3d7bb6bc360cbf662300fe63fd/Images/Remove%20CD.png)

Then reboot VM.

---

### 3. Post-Installation Setup

## Docker

The official installation guide for different Linux distributions can be found here: https://docs.docker.com/engine/install
Install Docker using the official convenience script:

```bash
curl -fsSL https://get.docker.com | sh
```

Add your user to the docker group so you can run Docker commands without sudo:

```bash
sudo usermod -aG docker $USER
```

Verify the installation:

```bash
docker --version
docker compose version
```

Docker is not running yet, enable and start the service:
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify the docker status:
```bash
sudo systemctl status docker 
```
![](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/bbdb2cf3b30c5915dc26f1e7397fc747f5cd4bfa/Images/Docker%20status.png)

Docker is active and running, no further action is required.
---


## SSH

After Ubuntu Server is installed, perform the following steps.

## Update the system

```bash
sudo apt update && sudo apt upgrade -y
```

---

Check that the SSH service is running:

```bash
systemctl status ssh
```

If SSH is not running, enable and start it:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

## First SSH remote access

On your local computer or in the Termius mobile app, create a new SSH connection. Enter your Ubuntu username at the server's IP address, and your password.
```bash
ssh username@server_ip
```
Example:
```bash
ssh mikemilenk@192.168.1.105
Password:
```
The first time you connect, you'll be prompted to accept the server's host fingerprint. Once accepted, you'll be logged into the Linux Server remotely.


Verify that you can connect without entering your password.

---

## Tailscale

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate your server:

```bash
sudo tailscale up
```

Follow the URL displayed in the terminal to sign in and authorize the device.

---

You can SSH into the server using its Tailscale IP address:
```bash
ssh user@tailscale_ip
```
Example:
```bash
ssh mikemilenk@100.90.80.70
Password:
```

This works the same as regular SSH, but over the secure Tailscale network, allowing you to access the server from outside your local network.

---

## 4. Optional: Install QEMU Guest Agent (if not selected earlier)

Inside Ubuntu:
```bash id="guest_agent"
sudo apt update
sudo apt install qemu-guest-agent -y
sudo systemctl enable --now qemu-guest-agent
```

This improves Proxmox integration (IP display, shutdown control, etc.).

---
