# Deploying a Linux Server (Ubuntu 24.04) on Proxmox VE

This guide explains how to create and install an Ubuntu Server virtual machine on Proxmox VE. This VM will later be used to run services like Immich or other self-hosted applications.

---

## 0. Download Ubuntu Server ISO

First, download the Ubuntu Server image:

* Go to the official Ubuntu website: https://ubuntu.com/download/server
* Download the latest **LTS version (recommended)** I installed Ubuntu 24.04.4-live-server-amd64

After downloading, upload the ISO to Proxmox:

```text id="iso_upload"
PVE → local (storage) → ISO Images → Upload
```
![Uploading ISO](https://github.com/MikeMilenk/Immich-deployment/blob/ff3d949471ce37fbe5e0a1647dea83eca7c39248/images/Download%20ISO%20Images.png)

---

## 1. Create a New VM

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

## 2. OS Setup

* Storage **local**
* Select **Use CD/DVD disc image file (ISO)**
* Choose the uploaded **Ubuntu Server ISO**
* Guest OS type:
  * Type: **Linux**
  * Version: **Ubuntu (latest)**

Click **Next**
![OS](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/OS.png)

---

## 3. System Configuration

Recommended settings:

* **Display:** Default
* **Machine:** q35
* **BIOS:** OVMF (UEFI) *(recommended for modern setups)*
* Enable **QEMU Agent** (can be installed later in OS)

Click **Next**
![System](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/System.png)

---

## 4. Disk Configuration

Here you select where the VM disk will be stored.

* **Storage:** select your ZFS pool (`immich` in my case)
* **Disk size:** I allocated 150 GB (should be more than enough for running Ubuntu server only)
* **Bus/Device:** VirtIO SCSI (recommended)

Click **Next**
![Disks](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Disks.png)

---

## 5. CPU and Memory

* **CPU:** 2–4 cores (depends on workload)
* **Memory:** 8 GB minimum for Ubuntu Server with Immich installed. 16 GB is better. Note: 7812 MiB is ~8192 MB or 8 GB

Click **Next**
![CPU](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/CPU.png)
![Memory](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Memory.png)

---

## 6. Network Configuration (leave defaults)

* **Bridge:** `vmbr0`
* Model: **VirtIO (paravirtualized)**

Click **Next**
![Network](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Network.png)

---

## 7. Finish Setup

Review settings and click **Finish**.
![Confirmation](https://github.com/MikeMilenk/Deploying-Linux-Server/blob/7c23515ea0c22fe5fd082d75d88028b5d6bacd1d/Images/Confirmation.png)

---

## 8. Start Installation

1. Select the VM
2. Click **Start**
3. Open **Console**
4. Follow Ubuntu installation steps:
   * Language
   * Keyboard
   * Network (DHCP usually fine)
   * Disk installation (use default guided setup)
   * Create user and password

---

## 9. Post-Installation (Important)

After installation, remove ISO:

```text id="iso_remove"
VM → Hardware → CD/DVD Drive → Do not use any media
```

Then reboot VM.

---

## 10. Optional: Install QEMU Guest Agent (if not selected earlier)

Inside Ubuntu:
```bash id="guest_agent"
sudo apt update
sudo apt install qemu-guest-agent -y
sudo systemctl enable --now qemu-guest-agent
```

This improves Proxmox integration (IP display, shutdown control, etc.).

---

## Result

You now have a working Ubuntu Server VM running on Proxmox, ready for deploying services like Immich, Docker, or other self-hosted applications.
