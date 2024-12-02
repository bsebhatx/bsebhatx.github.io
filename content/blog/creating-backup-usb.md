---
title: "Creating Backup USB"
date: 2024-11-29T20:12:31-08:00
---

{{< youtube pLcALHjC1-g >}}

This guide outlines the steps to create an Arch Linux USB drive with persistent storage, encryption, and the KDE Plasma desktop environment. It uses the archuseriso project from [https://github.com/laurent85v/archuseriso](https://github.com/laurent85v/archuseriso) It also includes NVIDIA GPU support.

I'm using the SSK USB 3.2 Gen 2 SSD drive (Amazon link: [https://a.co/d/4trb0kQ](https://a.co/d/4trb0kQ).

REQUIRED: The `archuseriso` package. I can install it from the Arch Linux aur using the `yay` tool:
```bash
yay -Syu archuseriso
```
---

## 1. Create the Arch Linux ISO with KDE and NVIDIA Graphics

I'll create a directory in `/opt`:
```bash
sudo mkdir /opt/aui
```

I'll use the `/opt/aui` directory to create the ISO file.

The `archuseriso` command `aui-mkiso` will create a customized Arch Linux ISO.
```bash
sudo aui-mkiso kde --verbose --graphics=nvidia
```

- `kde`: This is the profile. It will create an ISO with the KDE Plasma desktop environment.
- `--graphics=nvidia`: Adds NVIDIA drivers for my laptop's GPU.
- `--verbose`: Enables detailed output for troubleshooting.

When I ran this today (Nov 30, 2024), I got an error about a missing target:

```bash
error: target not found: libva-vdpau-driver
```

That's a driver for the [Video Decode and Presentation API for Unix (VDPAU)](https://en.wikipedia.org/wiki/VDPAU). According to the Arch Linux Wiki on [Hardware video accelleration](https://wiki.archlinux.org/title/Hardware_video_acceleration):

*The code for libva-vdpau-driver has not been touched for years. As a result, it crashes when running VLC or OBS with recent versions of the NVIDIA driver. If you need a translation layer for NVIDIA drivers, use libva-nvidia-driver instead.*

I can ignore this package, and rerun the ISO creation, because it's not essential. I'll be installing NVIDIA drivers with the `--graphics=nvidia` option.

First, I'll edit the packages configuration for the KDE profile I'm using:
```bash
sudo vim /usr/share/archuseriso/profiles/kde/packages.x86_64
```

I'll comment out the line mentioning `libva-vdpau-driver`, and save it.

There will probably be a process running at `/opt/aui/work/x86_64/airootfs/proc`. I'll unmount:
```bash
sudo umount work/x86_64/airootfs/proc
```

And delete the `work` directory that was created:

```bash
sudo rm -fr work/
```

I'll rerun the command:

```bash
sudo aui-mkiso kde --verbose --graphics=nvidia
```

After a couple of minutes, this produces an ISO file in the `out` directory. I'll use that on my USB drive.

---

## 2. Plug in Your USB Drive

Insert your USB drive into your computer.

Verify the USB drive is detected and confirm the correct device name (something like `/dev/sda` or `/dev/sdb`):

```bash
lsblk
```

---

## 3. Verify the Partition Table Type

Check the partition table on the USB drive:

```bash
sudo fdisk -l /dev/sda
```

Look for `Disklabel type: dos` (for MBR) or `Disklabel type: gpt`.

If the drive uses MBR, I can convert it to GPT using the `parted` command.

I'll start `parted`:
```bash
sudo parted /dev/sda
```

Then, create a new GPT partition table:
```bash
mklabel gpt
```

And, finally, I'll exit `parted`:
```bash
quit
```

---


## 4. Install Arch Linux on the USB

Use the `aui-mkusb` command to install the Arch Linux ISO on the USB drive. 

```bash
    sudo aui-mkusb out/aui-kde*.iso /dev/sda --encrypt --size-cow=200G
```

This will:

- Create three partitions (including an EFI partition).
- Enable encryption for the system.
- Allocate 200 GB for the persistent copy-on-write (COW) partition.

---

## 5. Verify Partitions

List the block devices again to confirm the three partitions created by `aui-mkusb`.

```bash
lsblk
```

You should see:

1. EFI partition (e.g., `/dev/sda1`).
2. Root filesystem partition.
3. Persistent COW partition.

---

## 6. Create a Fourth Partition For Backup Storage
I want to create a fourth partition that will be encrypted. I'll use that for backing up files.

Use `cfdisk` to allocate the remaining space as a fourth partition.

1. Launch `cfdisk`:
```bash
sudo cfdisk /dev/sda
```

2. Select the free space.
3. Create a new partition using the remaining space.
4. Write changes and exit.

---

## 7. Initialize the Fourth Partition with LUKS (Encrypt It)

Format the newly created fourth partition (`/dev/sda4`) as a LUKS-encrypted partition:

```bash
    sudo cryptsetup luksFormat /dev/sda4
```

This is using all the default options. The Arch Linux Wiki has a section describing the different options at [https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encryption_options_for_LUKS_mode](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encryption_options_for_LUKS_mode). For instance, you can increase the time spent processing the passphrase from the default 5000 miliseconds (5 seconds) to 8000 by adding the option `--iter-time 8000`.

I set a passphrase for the encrypted partition. If I lose it, I won't be able to decrypt it. So it should be something I can remember.

---

## 8. Open the Encrypted Partition

Map the encrypted partition to `/dev/mapper/cryptusb`:

```bash
    sudo cryptsetup open /dev/sda4 cryptusb
```

---

## 9. Format the Encrypted Partition

Format the unencrypted mapped device with the `ext4` filesystem:

```bash
sudo mkfs.ext4 /dev/mapper/cryptusb
```

---

## 10. Verify the Partition Structure

Check the block devices again to confirm the setup:

```bash
lsblk
```

The encrypted partition should appear as `/dev/mapper/cryptusb`.

---

## 11. Mount the Encrypted Partition

Mount the unencrypted partition to `/mnt/cryptusb`, creating the directory if it doesn't exist:

```bash
sudo mount --mkdir /dev/mapper/cryptusb /mnt/cryptusb/
```

---

## 12. Write a Test File

Create a test file in the mounted partition to verify functionality:

```bash
sudo vim /mnt/cryptusb/secret.txt
```

---

## 13. Unmount the Partition

Unmount the encrypted partition when finished:

```bash
sudo umount /mnt/cryptusb
```

---

## 14. Close the Encrypted Partition

Remove the mapping for the encrypted partition:

```bash
sudo cryptsetup close cryptusb
```
---

## 15. Reboot To USB Drive

Now, to test it out, I can reboot my laptop and boot from the USB drive. It should have Arch Linux installed. It has a user account called `live`, but there's not password for it. I can set it with the `passwd` command.

Then, I can make changes to the system, and it will persist on the USB drive. If I'm having trouble with my laptop, I can boot from the USB drive and access the laptop hard drive by using the [arch-chroot](https://man.archlinux.org/man/arch-chroot.8) script.

So, for example, if my root partition is unencrypted and on the block device `/dev/nvme0n1p2`, I can mount it to the `/mnt/crypthd` directory:
```bash
sudo mount --mkdir /dev/nvme0n1p2 /mnt/crypthd
```

My laptop's hard drive is encrypted, so I can decrypt it and map it:
```bash
sudo cryptsetup open /dev/nvme0n1p2 crypthd
```

Then mount it:
```bash
sudo mount --mkdir /dev/mapper/crypthd /mnt/crypthd
```

When it's mounted, I can use the script `arch-chroot` to change my root to the laptop's root filesystem:
```bash
sudo arch-chroot /mnt/crypthd
```

## Final Notes

- Ensure the USB drive is correctly identified (e.g., `/dev/sda`) before executing commands to avoid overwriting another device.
- The process assumes the USB drive starts with an MBR partition table and is converted to GPT as part of this guide.
- The `aui-mkusb` command handles creating the EFI partition and other required partitions for the Arch Linux installation.