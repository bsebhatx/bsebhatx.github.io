---
title: Lab 01 - KVM, QEMU, and libvirt
type: docs
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
