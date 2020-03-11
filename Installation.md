# Arch Linux Installation Guide

## Partitioning and Formatting the Hard Drive

See the list of the disks in the device, if the target drive is not GPT then convert
it with *gdisk*<br />
`fdisk -l`

Open the partition manager in the disk you want to install<br />
`cfdisk /dev/sdx`

Create the partitions using the tool then hit enter on [Write]

- Create a partition of [550MiB](https://superuser.com/a/1310938) for EFI with `EFI System` as type
- Create one partition of for swap with `Linux swap` as type and space according to your [requirements](https://itsfoss.com/swap-size/)
- Create another partition for your system with `Linux filesystem` as type

Format the EFI partition<br />
`mkfs.fat -F32 /dev/sdx1`

Format the swap partition<br />
`mkswap /dev/sdx2`

Format the system partition<br />
`mkfs.ext4 /dev/mapper/arch-root`

Activate the swap partition<br />
`swapon /dev/sdaX`

## Installing the System

`mkdir -p /mnt/home`

Mount the root on /mnt<br />
`mount /dev/mapper/arch-root /mnt`

Mount the root on /mnt<br />
`mount /dev/mapper/arch-home /mnt/home`

To install the system ont root partition internet connection is needed, for cable
connections use `dhcp` and for wifi use `wifi-menu`

Use pacstrap to install the base system into your mount point<br />
`pacstrap /mnt base base-devel linux linux-firmware`

Install shell, editor and packages for wifi connection<br />
`pacman -S zsh vi vim iw dialog wpa_supplicant`

Create a boot directory for the EFI partition<br />
`mkdir /mnt/boot`

Mount EFI patition on boot directory<br />
`mount /dev/sdx1 /mnt/boot`

Enter the system's partition<br />
`arch-chroot /mnt`

Create a password to the root user<br />
`passwd`

Create a group and user<br />
`groupadd youruser && useradd -mg your_user -G wheel -s /bin/zsh your_user`

Change your user password<br />
`passwd your_user`

Enable wheel group<br />
`visudo`

Set language by unncomenting your it on locale.gen<br />
`vim /etc/locale.gen`

Create the locale file and config language<br />
`locale-gen`
`echo LANG=en_US.UTF-8 > /etc/locale.conf`
`export LANG=en_US.UTF-8`

Create a link between the timezone file and localtime file and set hwclock to store UTC<br />
`ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime`<br />
`hwclock --systohc --utc`

Choose your hostname<br />
`echo hostname > /etc/hostname`

We need to edit `/etc/mkinitcpio.conf` to provide support for lvm2.
Edit the file and insert lvm2 between block and filesystems like so:

`HOOKS="base udev ... block lvm2 filesystems"`

Generate an init file which grub uses to load linux<br />
`mkinitcpio -p linux`

Install systemd-boot to the EFI system partition:

`bootctl install`

Now we need to create a boot entry. Edit `/boot/loader/loader.conf`:

```
default  arch
timeout  4
editor   0
```

Create the arch entry by editing `/boot/loader/entries/arch-lvm.conf`:

```
title          Arch Linux (LVM)
linux          /vmlinuz-linux
initrd         /initramfs-linux.img
options        root=/dev/mapper/lvm-root rw
```

Disconnect from chroot session<br />
`exit`

Generate fstab file based on the hard drive<br />
`genfstab /mnt >> /mnt/etc/fstab`

Unmount the hard drive but unmount everything mounted on top of it, as the boot
directory<br />
`umount /mnt/boot`<br />
`umount /mnt`

Now reboot the system and check if everything is working.
