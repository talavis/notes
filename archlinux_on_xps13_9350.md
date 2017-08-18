# Installation of Archlinux on a Dell XPS 13 9350 
## Introduction
The following are my notes on installing Archlinux with full disc encryption on a XPS13. The computer had an i5, 1080p screen, 256GB SSD and 8G RAM.

I recommend taking a look at the [XPS13](https://wiki.archlinux.org/index.php/Dell_XPS_13_(2016)) and [beginner's guide](https://wiki.archlinux.org/index.php/beginners&#39;_guide) pages in theArchlinux wiki. Most information listed here comes from the wiki, and I sincerely thank all contributors to it.

Note that if you follow the instructions, you will delete all old data on the computer. 

## Preparations
Get the Archlinux ISO from the [homepage](http://www.archlinux.org/). Install it to a USB stick or similar using e.g.: 
`dd if=archlinux-2016.03.01-dual.iso of=/dev/sdx`

Boot the XPS13 and go to BIOS (F12 at boot) and turn off Secureboot to be able to boot from the USB stick with Archlinux. In order to be able too find the hard drive in the XPS13, you will need to change the SATA controller from "RAID On" to "AHCI".

## Installation
Boot from the USB stick and confirm that efivars are working; check if the command `ls /sys/firmware/efi/efivars` gives any output.

### Partitions
Partition the hard drive; I decided to reset everything and create two partitions; one 512 MB EFI partition for /boot and one partition for an encrypted lvm container for the rest. I used gdisk, but you can as well use parted. Note that the disk is attached via nvme, and is thus called nvme0n1 instead of sda

#### Creating the partitions:

~~~~
	gdisk /dev/nvme0n1
	ID: 
	Start (sector):
	End (sector): +512M
	Code: EF00
	ID: 2
	Start (sector):
	End (sector):
~~~~

Creating a vfat file system for the EFI partition:

`	mkfs.vfat -F32 /dev/nvme0n1p1`

#### Encrypted LVM container

Creating the encrypted lvm container (I chose 40 GB for root as I tend to have a lot of programs, and 16 GB for swap as I sometimes need a lot of things open; both can be made smaller):
~~~~
	modprobe dm-crypt
	
	cryptsetup -c aes-xts-plain64 -y -s 512 luksFormat /dev/nvme0n1p2
	cryptsetup luksOpen /dev/nvme0n1p2 lvm
	lvm pvcreate /dev/mapper/lvm
	lvm vgcreate vgroup /dev/mapper/lvm
	lvm lvcreate -L 40GB -n root vgroup
	lvm lvcreate -C y -L 16GB -n swap vgroup
	lvm lvcreate -l 100%Free -n home vgroup
~~~~

Making filesystems for the new "partitions":
~~~~
    mkfs.ext4 /dev/mapper/vgroup-root
	mkfs.ext4 /dev/mapper/vgroup-home
	mkswap /dev/mapper/vgroup-swap
~~~~

### Mount partitions

~~~~
    mount /dev/mapper/vgroup-root /mnt
    mkdir -p /mnt/boot
    mount /dev/nvme0n1p1 /mnt/boot
    mkdir -p /mnt/home
    mount /dev/mapper/vgroup-home /mnt/home
    swapon -va /dev/mapper/vgroup-swap
~~~~

### Connect to the internet

Connect to a wireless connection point using e.g. wifi-menu.

### Base installation

Install the basic programs:
`pacstrap /mnt base base-devel`
Generate a fstab:
`genfstab -p -U /mnt &gt;&gt; /mnt/etc/fstab`
Chroot to the new installation:
`arch-chroot /mnt`

### Basic settings

Set hostname:
`echo hostname > /etc/hostname`
Set timezone:
`ln -s /usr/share/zoneinfo/Your/Timezone /etc/localtime`

#### Set locale preferences:

Append your locale to `/etc/locale.conf`, e.g.:
~~~~
      LANG="en_GB.UTF-8"
      LC_COLLATE="C"
~~~~
Uncomment the corresponding line in `/etc/locale.gen`, e.g. `en_US.UTF-8`. Then run `locale-gen` to generate the locale.

#### Configure mkinitcpio.conf
Edit `/etc/mkinitcpio.conf` and add the needed hook for encryption as well the nvme to modules:
~~~~
    MODULES="... nvme
...
    HOOKS=(... sata encrypt lvm2 filesystems ...)
~~~~
And afterwards run:
`mkinitcpio -p linux`

### Bootloader
GRUB is not compatible with nvme disks, but bootd is a good replacement. Just use `bootctl install`

To add the support for the encrypted lvm, open `/boot/loader/entries/arch.conf` and add the line:
`options cryptdevice=/dev/nvme0n1p2:vgroup root=/dev/mapper/vgroup-root quiet rw`

The system should now be ready for reboot. Remove the USB stick and do so.

## Post-installation

Install relevant packages. It might be wise to look into e.g. swappiness, intel-ucode and e.g. sddm for graphical login.
