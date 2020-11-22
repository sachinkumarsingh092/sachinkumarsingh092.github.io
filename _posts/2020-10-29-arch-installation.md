---
title: Installing Arch Linux
tags: [Arch, Linux]
style: fill
color: dark
description: Guide to installing Arch Linux from scratch.
---

![Arch Linux](../assets/arch.jpg "Arch Linux")
I've written this blog so as to easily import all my current configurations when migrating to a new system.
For the official installation guide, head over to [this arch wiki](https://wiki.archlinux.org/index.php/installation_guide).
It's easy to mess up your system as you'll have root privilages during the installation. Search the commands if you are not sure what it does. [Arch wiki](https://wiki.archlinux.org/) is the best resource at your disposal. Use it wisely ;-)

> I'll use a USB flash drive to make an installation medium from the ISO image and [rufus](https://rufus.ie/) to create bootable USB flash drives.

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
| `[SWAP]`              | `/dev/sda`            | Linux swap            | At least 512 MiB      |
| `/home`               | `/dev/sda`            | Home partition        | At least 20 GB        |
| `/boot`               | `/dev/sdb`            | EFI system partition  | At least 260 MiB      |
| `/`                   | `/dev/sdb`            | Linux x86-64 root (/) | At least 30 GB        |


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

#### Format the partitions and mount filesystems
Set up the file system of the root partition:
```zsh
# mkfs.ext4 /dev/sdbX
```
Mount the decrypted partition into `/mnt':
```zsh
# mount /dev/sdbX /mnt
```
Mount the  boot partition under `/mnt/boot':
```zsh
# mkfs.vfat /dev/sdbY
# mkdir /mnt/boot
# mount /dev/sdbY /mnt/boot
```

Mount all other partitions you may have on your system under `/mnt' (future root directory).

Even a small SWAP partition can be useful (atleast to avoid warnings). Once you have set up such a partition, you can format and activate it like this:
```zsh
# mkswap /dev/sdaX
# swapon /dev/sdaX
```
genfstab will later detect mounted file systems and swap space.

## Installation and configuration
Install the base operating system components:
```zsh
# pacstrap /mnt base base-devel linux linux-firmware man-db man-pages texinfo vim
```
Set up the file system table for the new partitions:
```zsh
# genfstab -U /mnt >> /mnt/etc/fstab
```

Go into the newly setup root system:
```zsh
# arch-chroot /mnt
```

Set up the timezone and generate `/etc/adjtime:
```zsh
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
# hwclock --systohc
```

Setup the locale:
```zsh
# vim /etc/locale.gen        
Uncomment `en_US.UTF-8 UTF-8`
# locale-gen
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Setup the system's host name:
```zsh
# echo "MYNAME" > /etc/hostname
# echo "127.0.0.1        localhost"                      >> /etc/hosts
# echo "::1              localhost"                      >> /etc/hosts
# echo "127.0.1.1        MYNAME.localdomain      MYNAME" >> /etc/hosts\
```

Setup the kernel module by making sure that the `HOOKS` contain the following modules (in this order): `base udev autodetect keyboard keymap modconf block encrypt filesystems fsck`. Then reload the kernel:
```zsh
# vim /etc/mkinitcpio.conf
# mkinitcpio -p linux
```

Set the root password:
```zsh
# passwd
```

Install the necessary packages and enable their systemd services:
```zsh
# pacman -S intel-ucode iw iwd wpa_supplicant dialog grub nano dhcpcd efibootmgr parted networkmanager network-manager-applet

# systemctl enable NetworkManager
```
Use GRUB for the bootloader. Note that the GPT table is only for the DEVICE (not partition). So if you didn't change the partition table of the device, you don't need to worry about that. Then open `/etc/default/grub` and make the following corrections:
```zsh
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# vim /etc/default/grub
 --> Set the timer to 1, so it doesn\'t wait to long.
 --> Remove "quiet" from `GRUB_CMDLINE_LINUX_DEFAULT`.
 --> GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:cryptroot".
 --> uncomment `GRUB_ENABLE_CRYPTODISK=y`.
# grub-mkconfig -o /boot/grub/grub.cfg
```

Exit the chroot environment:
```zsh
# exit
```

Unmount the partitions and reboot the system:
```bash
# umount /mnt/boot
# umount /mnt
# systemctl reboot
```

After the reboot (and decrypting the root partition), login as `root` with the password that you set.

For Wifi use the same 'iwctl' commands at the start.

Disable the case-speaker (if it bothers): After the first boot (where you have direct access to the kernel, not through a chroot environment), you can stop the internal case speaker from beeping with this command:
```zsh
# modprobe -r pcspkr
# echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf
```

Define the system users:
```zsh
# useradd --create-home -s /bin/bash USERNAME
# passwd USERNAME
```

Enable Sudo for the users (where necessary). 
> NOTE that the first time you run `sudo ...` you will get a warning, but that occurs only at the first time. It will not show up on the next runs of sudo.
```zsh
# vim /etc/sudoers
--> Add "USERNAME ALL=(ALL) ALL" after you see a similar line
    for `root`: (replacing USERNAME with the name of the user):
```

Install other basic necssary software:
```zsh
# pacman -S openssh gdb libcups git xarchiver ntfs-3g firefox wget dosfstools nuspell lzip rsync ifplugd aspell aspell-en exfat-utils fuse-exfat adobe-source-han-sans-otc-fonts otf-ipafont sshfs noto-fonts-extra  

# systemctl enable sshd
```

If you have existing SSH keys in `~/.ssh`, run `ssh-add` to add them to the authentication agent.

