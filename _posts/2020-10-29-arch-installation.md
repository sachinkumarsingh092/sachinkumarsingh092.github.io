---
title: Installing Arch Linux
tags: [Arch, Linux]
style: fill
color: sucess
description: Guide to installing Arch Linux from scratch.
---

![Arch Linux](../assets/arch.png "Arch Linux")
I've written this blog so as to easily import all my current configurations when migrating to a new system.
For the official installation guide, head over to [this arch wiki](https://wiki.archlinux.org/index.php/installation_guide).

> I'll use a USB flash drive to make an installation medium from the ISO image. I use [rufus](https://rufus.ie/) create bootable USB flash drives.

## Pre-installation
In the boot configuration (hardware), make sure that the bios only boots into UEFI mode, not "legacy" mode, the modern GRUB bootloader (or other bootloaders) only work on UEFI. To verify the boot mode, list the efivars directory:
```zsh
# ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode.

> If you also have a discrete graphic card, it can cause problems in the boot or shutdown. So try the different configurations    (`only integrated`, `only discrete`, `hybrid` or `automatic`). Currently the `only integrated` suits me best.

#### Wifi
If you don't have an Ethernet port to connect with the internet, follow these steps to set up the wifi.

`iwctl` starts a new prompt for `iwd` (iNet wireless daemon), next commands are within that prompt:
- `device list`: to list existing wifi devices.
- `station wlan0 scan`: it will scan all networks.
- `station wlan0 get-networks`: list all found networks.
- `station device connect SSID`: set SSID to your network name.
- `quit`.                 
- To make sure the network is setup, run `ping archlinux.org`.

#### Disk-partition
I'll use [GNU Parted](https://wiki.archlinux.org/index.php/Parted) to create partion tables.

Use `lsblk` to check the block devices on the disks:
```zsh
# lsblk
```

I have both SSD (128 GB, /dev/sdb) and HDD (1 TB, /dev/sda) drivers and will make the partions to boot in UEFI using [GPT](https://wiki.archlinux.org/index.php/Partitioning#GUID_Partition_Table) partioning scheme, as given in the table below:

| Mount Point Name      | Partion               | Partion Type          | Suggested Size        |
|:----------------------|:----------------------|:----------------------|:----------------------|
| `[SWAP]`              | `/dev/sda`**a**       | Linux swap            | More than 512 MiB     |
| `/home`               | `/dev/sda`**b**       | Home partiotion       | At least 20 GB        |
| `/boot/efi`           | `/dev/sdb`**c**       | EFI system partition  | At least 260 MiB      |
| `/`                   | `/dev/sdb`**d**       | Linux x86-64 root (/) | At least 30 GB        |

where a, b, c and d are the serial number for the partions.

Start GNU Parted on device. 
```zsh
# parted /dev/sda 
```
To list existing parition tables use:
```zsh
(parted) print 
```
To delete any existing partitions use:
```zsh
(parted ) rm NUMBER
```
Set a new GUID Partition table (GPT):
```zsh
(parted) mklabel gpt
```
Make the partion for bootable efi system and other partions:
```zsh
(parted) mkpart primary fat32 1MiB 551MiB  # Create the boot partition.    
(parted) set 1 esp on                      # Set the "EFI System Partition" (esp) flag.    
(parted) mkpart primary ext4 551MiB 100%   # Set the single partition (all the remaining size)
```
Another way to make partions using starting and ending block points:
```zsh
(parted) mkpart "root partition" ext4 261MiB 20.5GiB    
(parted) mkpart "swap partition" linux-swap 20.5GiB 24.5GiB
(parted) mkpart "home partition" ext4 24.5GiB 100%    
(parted) quit
```
