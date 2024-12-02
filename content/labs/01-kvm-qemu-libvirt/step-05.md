---
title: "Step 05: Use virt-manager"
type: docs
breadcrumbs: true
---

The `virt-manager` graphical application provides a user-friendly interface to manage virtual machines, networks, and storage pools. In this step, you'll use it to modify network settings and create a storage pool.

## Edit Default Virtual Network DHCP Settings

1. Open `virt-manager` from your desktop application menu or by running the following command in a terminal:
   ```bash
   virt-manager
   ```

2. Select the **default** virtual network from the interface and open its **Properties**.

3. Navigate to the **IPv4 Configuration** tab and modify the DHCP range:
   - Change the **Start** address from `192.168.122.2` to `192.168.122.10`.

   This configuration reserves the range `192.168.122.2` to `192.168.122.9` for static IP addresses, which can be manually assigned to virtual machines.

4. Save the changes and restart the network to apply the updated settings.

## Create a Storage Pool for ISO Files

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