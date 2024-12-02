---
title: "Step 02: Install Packages"
type: docs
breadcrumbs: true
---

I'll need to install the software packages for emulating hardware and networks.

## Update Linux Repositories

Before installing new packages, it is essential to update your Linux repositories to ensure you are getting the latest versions of the packages.

### Arch Linux
Run the following command:
```bash
sudo pacman -Syu
```

### Ubuntu
Run the following commands:
```bash
sudo apt update
sudo apt upgrade
```

### Fedora
Run the following command:
```bash
sudo dnf update
```

## Install Virtualization Packages

Install the required packages for virtualization, including QEMU, libvirt, virt-manager, and any necessary networking utilities for virtual bridges.

### Arch Linux
Run the following command:
```bash
sudo pacman -S qemu libvirt virt-manager bridge-utils
```

### Ubuntu
Run the following command:
```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

### Fedora
Run the following command:
```bash
sudo dnf install @virtualization
```