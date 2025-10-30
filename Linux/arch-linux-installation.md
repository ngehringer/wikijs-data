---
title: Arch Linux Installation
description: 
published: true
date: 2025-10-30T08:55:15.982Z
tags: 
editor: markdown
dateCreated: 2025-10-30T08:33:52.643Z
---

# Arch Linux Installation

In the process of installing arch linux as my secondary system on my main computer, I am documenting every caveat and specifics that caused issues.
Installing arch (with secure boot) is not an easy process and mistakes can easily be made. The specifics here mainly apply on my use case but can be useful for anyone wanting to install Arch on their own.

## Table of Contents

1. [Partitionning (dual booting with windows)](#partitionning)
2. [Configuring the bootloader](#configuring-the-bootloader)

## <a name="partitioning"></a>Partitioning (dual booting with windows)

## <a name="configuring-the-bootloader"></a>Configuring the bootloader

/boot/refind_linux.conf
```
"Boot using default options"     "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap"
"Boot using fallback initramfs"  "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap initrd=boot\initramfs-%v-fallback.img"
"Boot to terminal"               "root=PARTUUID=ff6416fd-9462-4bd4-94f5-133787fae1a3 rw add_efi_memmap systemd.unit=multi-user.target"
```

## Sources

https://github.com/Foxboron/sbctl?tab=readme-ov-file
https://wiki.archlinux.org/title/REFInd
https://github.com/archlinux/archinstall?tab=readme-ov-file#how-to-dual-boot-with-windows
https://wiki.archlinux.org/title/Dual_boot_with_Windows
https://wiki.archlinux.org/title/Hyprland
https://wiki.archlinux.org/title/Installation_guide
https://wiki.archlinux.org/title/Archinstall
https://itsfoss.com/install-yay-arch-linux/