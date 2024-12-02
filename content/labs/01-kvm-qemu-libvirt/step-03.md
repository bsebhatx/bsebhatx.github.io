---
title: "Step 03: Setup libvirtd"
type: docs
breadcrumbs: true
---

The `libvirtd` service is a monolithic, all-in-one daemon responsible for managing virtual machines, storage, and networking. While libvirt is gradually moving towards a modular architecture with separate services for each function, the monolithic `libvirtd` service is easier to use in lab environments due to its simplicity and unified management approach.

## Check if libvirtd is Running

To verify whether the `libvirtd` daemon is running, use the following command:

```bash
sudo systemctl status libvirtd
```

If the service is active, you are ready to proceed. If it is inactive or not running, continue to the next step.

## Start libvirtd if Not Running

If the `libvirtd` service is not running, start it with the following command:

```bash
sudo systemctl start libvirtd
```

If you plan to use virtualization frequently, you can enable the service to start automatically at boot:

```bash
sudo systemctl enable libvirtd
```

## Add User to the libvirt Group

For non-root users to manage virtual machines, they must be added to the `libvirt` group. Use the following command to add your user account:

```bash
sudo usermod -aG libvirt $USER
```

After adding yourself to the group, log out and log back in for the changes to take effect. Being part of the `libvirt` group grants permission to interact with virtualization resources without needing root privileges.