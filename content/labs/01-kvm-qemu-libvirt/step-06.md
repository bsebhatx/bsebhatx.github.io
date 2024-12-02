---
title: "Step 06: Create Virtual Machine"
type: docs
breadcrumbs: true
---

In this step, you'll create a Fedora Linux virtual machine with emulated hardware using `virt-manager`.

## Download Fedora Workstation ISO

1. Visit the Fedora Workstation download page: [Fedora Workstation Download](https://fedoraproject.org/workstation/download).
2. Download the Fedora Workstation ISO file and move it to the `/var/lib/libvirt/ISOs` directory:
   ```bash
   sudo mv ~/Downloads/<Fedora_ISO_File_Name> /var/lib/libvirt/ISOs
   ```

## Create a New Virtual Machine

1. Open `virt-manager`:
   ```bash
   virt-manager
   ```

2. Click **Create a New Virtual Machine** and follow these steps:
   - Select **Local install media (ISO image or CDROM)** and browse to the Fedora Workstation ISO in `/var/lib/libvirt/ISOs`.
   - Assign the following resources:
     - **Memory**: 4 GB (4096 MB)
     - **CPUs**: 2
     - **Storage**: 20 GB (default configuration)
   - Name the virtual machine **sysadmin** and begin the installation.

## Install Fedora Workstation

1. Follow the Fedora Workstation installation process:
   - Select the desired language and keyboard settings.
   - Configure the disk for automatic partitioning.
   - Set a hostname if prompted.
   - Begin the installation.

2. Once the installation is complete, reboot the virtual machine and create a user account named **vmadmin** during the first boot setup.

## Test Internet Access

1. Log in to the virtual machine using the **vmadmin** account.
2. Open a terminal and test internet access by running:
   ```bash
   ping -c 4 google.com
   ```

The virtual machine is using the default virtual network in NAT mode. NAT (Network Address Translation) allows the virtual machine to access the internet via the host machine. The VM is assigned a private IP address on the virtual network, and the host acts as a gateway for outbound internet traffic.

If the internet access is successful, the virtual machine setup is complete.