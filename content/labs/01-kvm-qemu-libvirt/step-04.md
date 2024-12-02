---
title: "Step 04: Use virsh"
type: docs
breadcrumbs: true
---

The `virsh` command-line tool interacts with the `libvirtd` service to manage virtual machines, storage pools, and virtual networks. It provides powerful functionality for administrators working in terminal-based environments.

## List Defined Virtual Networks

To view the defined virtual networks, use the following command:

```bash
virsh net-list --all
```

By default, `virsh` connects to the URI `qemu:///session`, which is a user-specific instance and does not display system-wide resources like virtual networks. To connect to the system instance, explicitly specify the URI:

```bash
virsh --connect=qemu:///system net-list --all
```

Alternatively, running `virsh` with `sudo` automatically changes the default URI to `qemu:///system`, which displays the system-wide networks.

## Set Default URI Environment Variable

To avoid specifying the URI every time, set the default URI in your shell configuration file. Add the following line to your `~/.bashrc` file:

```bash
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Reload the shell configuration:

```bash
source ~/.bashrc
```

Now `virsh` will default to the `qemu:///system` URI without needing additional flags.

## Start the Default Network

The default virtual network provided by libvirt may not be running. To start it, use the following command:

```bash
virsh net-start default
```

If you want the network to start automatically when the `libvirtd` service starts, enable it:

```bash
virsh net-autostart default
```