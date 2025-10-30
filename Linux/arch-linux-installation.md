---
title: Arch Linux Installation
description: 
published: true
date: 2025-10-30T20:23:40.737Z
tags: 
editor: markdown
dateCreated: 2025-10-30T08:33:52.643Z
---

# Arch Linux Installation

In the process of installing arch linux as my secondary system on my main computer, I am documenting every caveat and specifics that caused issues.
Installing arch (with secure boot) is not an easy process and mistakes can easily be made. The specifics here mainly apply on my use case but can be useful for anyone wanting to install Arch on their own.

This guide assumes that you have basic understanding of Linux and its installation process. It is **NOT** a complete guide to install arch.

This documentation is written with the goal being to install [Omarchy](https://omarchy.org/) on top of Arch so regarding archinstall options I mainly followed the documentation [here](https://learn.omacom.io/2/the-omarchy-manual/96/manual-installation).

Don't forget to check the sources at the end of this page if you feel lost or unsure of what to do.

## Table of Contents

1. [Prepare your system](#prepare)
2. [Partitionning (dual booting with windows)](#partitionning)
3. [Configuring the bootloader](#configuring-the-bootloader)
4. [Setup Secure Boot](#secure-boot)
5. [Install OMARCHY](#omarchy)
6. [(Optional) Install NVIDIA proprietary drivers](#nvidia-drivers)
7. [Sources](#sources)

## <a name="prepare"></a> Prepare your system

In order to dual boot Arch Linux with Windows, you have to configure the partitions manually.
First, create an empty partition on the same drive where Windows is installed. 

Before restarting, Windows hibernation is known to cause issues with dual booting so disable it using an administrator CMD prompt :
```
powercfg /H off
```

Then, boot on your BIOS and **DISABLE** Secure Boot as the Arch installation media is not signed. You can then boot on your install media.

Once inside Arch live ISO, configure you locale using loadkeys; *e.g. loadkeys fr*

Then, run the following commands :
```bash
# Ensures both core and extra are up to date
pacman -Sy

# Update archinstall to the latest version
pacman -S archinstall

# Run archinstall
archinstall
```

From now on if you want to install omarchy follow the Omarchy guide but **DON'T CONFIGURE DISK YET** (don't change bootloader since we're going to change it anyway), or configure accordingly to your desired configuration.

## <a name="partitioning"></a>Partitioning (dual booting with windows)

While configuring archinstall on Disk configuration, select manual configuration and select the drive where you'll be installing arch (the one where Windows is installed).
Select the empty partition, select the **ext4** filesystem and set is as the root mountpoint (/)
Then, select the partition labelled as boot, efi and mount it as **/boot/efi**

For omarchy specifically :
* Create a third ext4 partition of 1GiB mounting on /boot -> failure to do so will prevent your bootloader to find your kernel !
* Configure disk encryption using LUKS under Disk > Disk Encryption (follow the omarchy guide for this part, select your root partition when selecting which one to encrypt)

Once you are satisfied with your configuration, you can install arch.

## <a name="configuring-the-bootloader"></a>Configuring the bootloader

This is the tricky part, especially if you configured disk encryption.

Once the installation is finished, chroot on the installed system, install refind and run it
```bash
pacman -S refind

refind-install
```

It'll create its configuration file in /boot/refind_linux.conf. Delete it, as it is using the live ISO configuration and if you reboot now it won't boot.

Create the correct refind_linux.conf in /boot following the following examples depending if you used LUKS or not

### With LUKS
Scripting refind_linux.conf (change p2 accordingly to your available partitions)
```bash
BLOCK_DEVICE=/dev/nvme0n1
LUKS_UUID=$(blkid -s UUID -o value "${BLOCK_DEVICE}p2")
BOOT_OPTIONS="cryptdevice=UUID=${LUKS_UUID}:cryptlvm root=/dev/mapper/cryptlvm"

cat <<EOF >/mnt/boot/refind_linux.conf
"Boot with standard options"  "${BOOT_OPTIONS} rw loglevel=3"
"Boot to single-user mode"    "${BOOT_OPTIONS} rw loglevel=3 single"
"Boot with minimal options"   "ro ${BOOT_OPTIONS}"
EOF
```

### Without LUKS
/boot/refind_linux.conf where PARTUUID is the unique id attributed to your root partition (check with lsblk)
```
"Boot using default options"     "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap"
"Boot using fallback initramfs"  "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap initrd=boot\initramfs-%v-fallback.img"
"Boot to terminal"               "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap systemd.unit=multi-user.target"
```

You can now exit and reboot your machine and check if your boot configuratin is correct (hopefully). Don't forget to change your UEFI boot order or you'll boot on Windows.

## <a name="secure-boot"></a> Setup secure boot

To setup secure boot, you must first set your BIOS to setup mode. To do this, boot into your BIOS, go to secure boot configuration and delete enrolled keys (check a guide for your motherboard vendor).

Then, directly boot in arch and install [sbctl](https://github.com/Foxboron/sbctl)
```bash
pacman -S sbctl
```

Once sbctl is installed, basically follow sbctl guide to create a signing key, enroll it and sign your bootloader and your kernel :

RUN AS ROOT
```bash
# Check sbctl status
sbctl status

# Create signing keys
sbctl create-keys

# Enroll signing keys (add --microsoft because dual boot)
sbctl enroll-keys --microsoft

# You might have an error if you have tpm enabled. In this case, you can try with this command
sbctl enroll-keys --microsoft --tpm-eventlog

# Check that your keys are enrolled
sbctl status
```

Once your key is enrolled, you must sign your bootloader as well as your kernel

```bash
# Sign boot file
sbctl -s /boot/efi/EFI/BOOT/BOOTX64.EFI

# Sign refind bootloader
sbctl -s /boot/efi/EFI/refind/refind_x64.efi

# Sign kernel
sbctl -s /boot/vmlinuz-linux
```

Once done, reboot and enable Secure Boot. If you did everything correctly, you should be able to boot into Arch without issues.

## <a name="omarchy"></a>Install OMARCHY

To install Omarchy, run the following script
```bash
curl -fsSL https://omarchy.org/install | bash
```

## <a name="nvidia-drivers"></a> (Optional) Install NVIDIA proprietary drivers

## Sources

https://github.com/Foxboron/sbctl?tab=readme-ov-file
https://wiki.archlinux.org/title/REFInd
https://github.com/archlinux/archinstall?tab=readme-ov-file#how-to-dual-boot-with-windows
https://wiki.archlinux.org/title/Dual_boot_with_Windows
https://wiki.archlinux.org/title/Hyprland
https://wiki.archlinux.org/title/Installation_guide
https://wiki.archlinux.org/title/Archinstall
https://itsfoss.com/install-yay-arch-linux/
https://learn.omacom.io/2/the-omarchy-manual/96/manual-installation
https://www.reddit.com/r/archlinux/comments/15v7i7z/refind_boot_options_for_luks_partition_with_lvm/?show=original