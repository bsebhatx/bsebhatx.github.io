---
title: Lab 01 - KVM, QEMU, and libvirt
type: docs
next: 02-virtual-networks
breadcrumbs: true
---

## Introduction

Virtualization is a foundational technology in modern IT infrastructure. This lab focuses on setting up and understanding KVM, QEMU, and libvirt on a Linux system.

### What is KVM?

KVM (Kernel-based Virtual Machine) is a **type 1 hypervisor** because it is integrated directly into the Linux kernel. Unlike **type 2 hypervisors**, which run on top of an operating system (e.g., VMware Workstation, VirtualBox), **type 1 hypervisors** operate directly on the hardware for better performance. Examples of type 1 hypervisors include KVM and VMware ESXi, while examples of type 2 hypervisors include VMware Workstation and Oracle VirtualBox.

### What is QEMU?

QEMU is a versatile emulator that simulates hardware devices. When paired with KVM, QEMU can leverage hardware virtualization to run virtual machines at near-native speeds. While KVM handles the virtualization layer, QEMU provides the necessary emulation for devices and systems.

### What is libvirt?

libvirt is a toolkit that simplifies virtualization management. It abstracts complex virtualization operations and provides a consistent interface for managing virtual machines. Tools like `virsh` (a command-line utility) and `virt-manager` (a graphical utility) use libvirt to interact with virtual machines.

---

## Step 01: Check That System Can Use Virtualization

Before setting up KVM, ensure your system supports hardware virtualization.

### Check for Virtualization Support

Open a terminal and check the contents of `/proc/cpuinfo` for virtualization flags:

```bash
grep -E 'vmx|svm' /proc/cpuinfo
```

- **vmx**: Indicates Intel Virtualization Technology is supported.
- **svm**: Indicates AMD Secure Virtual Machine is supported.

If you see one of these flags, your system supports hardware virtualization.

### Enable Virtualization in the BIOS/UEFI

If no flags appear, virtualization might not be enabled in your BIOS/UEFI settings. To enable it:

1. Reboot your system and enter the BIOS/UEFI setup (commonly accessed by pressing a key like `F2`, `DEL`, or `ESC` during boot).
2. Look for an option related to **Virtualization Technology** (Intel VT-x) or **SVM Mode** (for AMD CPUs).
3. Enable the setting, save changes, and reboot.

If you're unsure how to access or modify these settings, consult your computer's documentation or search online for steps specific to your system's make and model.


## Step 02: Install Virtualization Packages
I'll need to install the software packages for emulating hardware and networks.

### Update Linux Repositories

Before installing new packages, it is essential to update your Linux repositories to ensure you are getting the latest versions of the packages.

#### Arch Linux
Run the following command:
```bash
sudo pacman -Syu
```

#### Ubuntu
Run the following commands:
```bash
sudo apt update
sudo apt upgrade
```

#### Fedora
Run the following command:
```bash
sudo dnf update
```

### Install Virtualization Packages

Install the required packages for virtualization, including QEMU, libvirt, virt-manager, and any necessary networking utilities for virtual bridges.

#### Arch Linux
Run the following command:
```bash
sudo pacman -S qemu libvirt virt-manager bridge-utils
```

#### Ubuntu
Run the following command:
```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

#### Fedora
Run the following command:
```bash
sudo dnf install @virtualization
```

## Step 03: Setup libvirtd Service

The `libvirtd` service is a monolithic, all-in-one daemon responsible for managing virtual machines, storage, and networking. While libvirt is gradually moving towards a modular architecture with separate services for each function, the monolithic `libvirtd` service is easier to use in lab environments due to its simplicity and unified management approach.

### Check if libvirtd is Running

To verify whether the `libvirtd` daemon is running, use the following command:

```bash
sudo systemctl status libvirtd
```

If the service is active, you are ready to proceed. If it is inactive or not running, continue to the next step.

### Start libvirtd if Not Running

If the `libvirtd` service is not running, start it with the following command:

```bash
sudo systemctl start libvirtd
```

If you plan to use virtualization frequently, you can enable the service to start automatically at boot:

```bash
sudo systemctl enable libvirtd
```

### Add User to the libvirt Group

For non-root users to manage virtual machines, they must be added to the `libvirt` group. Use the following command to add your user account:

```bash
sudo usermod -aG libvirt $USER
```

After adding yourself to the group, log out and log back in for the changes to take effect. Being part of the `libvirt` group grants permission to interact with virtualization resources without needing root privileges.


## Step 04: Use virsh Command Line Tool

The `virsh` command-line tool interacts with the `libvirtd` service to manage virtual machines, storage pools, and virtual networks. It provides powerful functionality for administrators working in terminal-based environments.

### List Defined Virtual Networks

To view the defined virtual networks, use the following command:

```bash
virsh net-list --all
```

By default, `virsh` connects to the URI `qemu:///session`, which is a user-specific instance and does not display system-wide resources like virtual networks. To connect to the system instance, explicitly specify the URI:

```bash
virsh --connect=qemu:///system net-list --all
```

Alternatively, running `virsh` with `sudo` automatically changes the default URI to `qemu:///system`, which displays the system-wide networks.

### Set Default URI Environment Variable

To avoid specifying the URI every time, set the default URI in your shell configuration file. Add the following line to your `~/.bashrc` file:

```bash
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Reload the shell configuration:

```bash
source ~/.bashrc
```

Now `virsh` will default to the `qemu:///system` URI without needing additional flags.

### Start the Default Network

The default virtual network provided by libvirt may not be running. To start it, use the following command:

```bash
virsh net-start default
```

If you want the network to start automatically when the `libvirtd` service starts, enable it:

```bash
virsh net-autostart default
```


## Step 05: Use virt-manager

The `virt-manager` graphical application provides a user-friendly interface to manage virtual machines, networks, and storage pools. In this step, you'll use it to modify network settings and create a storage pool.

### Edit Default Virtual Network DHCP Settings

1. Open `virt-manager` from your desktop application menu or by running the following command in a terminal:
   ```bash
   virt-manager
   ```

2. Select the **default** virtual network from the interface and open its **Properties**.

3. Navigate to the **IPv4 Configuration** tab and modify the DHCP range:
   - Change the **Start** address from `192.168.122.2` to `192.168.122.10`.

   This configuration reserves the range `192.168.122.2` to `192.168.122.9` for static IP addresses, which can be manually assigned to virtual machines.

4. Save the changes and restart the network to apply the updated settings.

### Create a Storage Pool for ISO Files

1. Open a terminal and create a directory to hold ISO files:
   ```bash
   sudo mkdir -p /var/lib/libvirt/ISOs
   sudo chmod 755 /var/lib/libvirt/ISOs
   ```

2. In `virt-manager`, go to the **Storage** tab and create a new storage pool:
   - Name the pool **ISOs**.
   - Set the **Target Path** to `/var/lib/libvirt/ISOs`.

3. Once the storage pool is created, mark it as active and set it to auto-start.

You can now download ISO files and place them in `/var/lib/libvirt/ISOs`. These files will be available in `virt-manager` when installing operating systems on virtual machines.


## Step 06: Create Virtual Machine

In this step, you'll create a Fedora Linux virtual machine with emulated hardware using `virt-manager`.

### Download Fedora Workstation ISO

1. Visit the Fedora Workstation download page: [Fedora Workstation Download](https://fedoraproject.org/workstation/download).
2. Download the Fedora Workstation ISO file and move it to the `/var/lib/libvirt/ISOs` directory:
   ```bash
   sudo mv ~/Downloads/<Fedora_ISO_File_Name> /var/lib/libvirt/ISOs
   ```

### Create a New Virtual Machine

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

### Install Fedora Workstation

1. Follow the Fedora Workstation installation process:
   - Select the desired language and keyboard settings.
   - Configure the disk for automatic partitioning.
   - Set a hostname if prompted.
   - Begin the installation.

2. Once the installation is complete, reboot the virtual machine and create a user account named **vmadmin** during the first boot setup.

### Test Internet Access

1. Log in to the virtual machine using the **vmadmin** account.
2. Open a terminal and test internet access by running:
   ```bash
   ping -c 4 google.com
   ```

The virtual machine is using the default virtual network in NAT mode. NAT (Network Address Translation) allows the virtual machine to access the internet via the host machine. The VM is assigned a private IP address on the virtual network, and the host acts as a gateway for outbound internet traffic.

If the internet access is successful, the virtual machine setup is complete.