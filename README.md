![Guide](https://user-images.githubusercontent.com/50109792/180651154-eae47f19-70ce-4e6f-aecd-4b78ced24f96.png)

<p align="center"
   <img src="https://user-images.githubusercontent.com/50109792/180651154-eae47f19-70ce-4e6f-aecd-4b78ced24f96.png" alt="Guide"/>
</p>

# Arch Linux MBR Installation Guide.

This is my personal guide. I recommend you to read the official [`installation wiki`](https://wiki.archlinux.org/index.php/Installation_guide). This guide will focus on `grub` and `MBR`. The purpose of this guide is to speed up the install process of `Archlinux`.

## Why Archlinux
The `Archlinux` distribution gives you the freedom to `do it yourself`. 

__NOTE__ that your kernel and initramfs are on the root file system, recovery after a crash may prove troublesome. 

They are many ways you can layout your partitions, I will focus on:

1. Type of hardware (*BIOS*)

2. The bootloader (*GRUB*)

3. I use my laptop for technical work, so the order,the type and the size of partitions matter for my use case.

4. I am just a speed enthusiast, like High Performance Computing and why not try to learn by doing it!

The __MAIN__ reason of installing vanilla linux(`Archlinux`) are:

  + __No Bloatware__ - Imagine `4GB` image for Windows, `2GB+` for Fedora or Ubuntu but Arch Linux approx. `700MB`. Softwares preinstalled that you will never use. 
  + __Performance & Security__ - Other distributions contain spyware you didn't install and are working in the background slowing your machine and making you quite uncomfortable.
  + __Learning__ - A better understanding of Linux.
  + __Minimalism__ - Keep It Simple Stupid (KISS)

## Pre-installation

Before installing, make sure to:

+ Read the [official wiki](https://wiki.archlinux.org/index.php/installation_guide). It is advisable to read that instead. I wrote this guide for myself.
+ Acquire an installation image from [here](https://www.archlinux.org/download/).
+ Verify signature.
+ Prepare an installation medium.
+ Boot the live environment.

## Create an Installation Medium
First we need to create an installation medium to boot from:

On Linux:

``````
# dd if=path_to_arch_iso of=/dev/sd* status=progress
``````
   + `progress` see periodic transfer statistics. 

---

# Installation Medium Boot
## Change Font Settings
If the terminal font is too small, which can happen if you have a high res display, then execute the following command:

Set the font size to: 

``````
# setfont lat4a-19 -m 8859-2
``````

## Connect to the internet

We need to make sure that we are connected to the internet to be able to install Arch Linux `base` and `linux` packages. Let’s see the names of our interfaces.

```
# ip addr
```

You should see something like this:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eno1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
3: wlo1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
		link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff permaddr 00:00:00:00:00:00
```

+ `eno1` is the wired interface  
+ `wlo1` is the wireless interface  

### Wired Connection

If you are on a wired connection, you can enable your wired interface by systemctl start `dhcpcd@<interface>`.  

```
# systemctl start dhcpcd@eno1
```

### Wireless Connection

If you are on a laptop, you can connect to a wireless access point using `iwctl` command from `iwd`. Note that it's already enabled by default. Also make sure the wireless card is not blocked with `rfkill`.

Scan for network.

```
# iwctl station wlo1 scan
```

Get the list of scanned networks by:

```
# iwctl station wlo1 get-networks
```

Connect to your network.

```
# iwctl -P "PASSPHRASE" station wlo1 connect "NETWORKNAME"
```

Ping archlinux website to make sure we are online:

```
# ping -c 5 8.8.8.8
``` 

+ `-c 5`    number of times to ping.
+ `8.8.8.8` Google DNS server.

If you receive Unknown host or Destination host unreachable response, means you are not online yet. Review your network configuration and redo the steps above.

## Update the system clock

Use `timedatectl` to ensure the system clock is accurate:

```
# timedatectl set-ntp true
```

To check the service status, use `timedatectl status`.

## Partition the disks

When recognized by the live system, disks are assigned to a block device such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use lsblk or fdisk.  The most common main drive is **sda**.

```
# lsblk
```

Results ending in `rom`, `loop` or `airoot` may be ignored.

In this guide, I'll create a one type of partition for the drive. A normal installation that is unencrypted:

### Unencrypted filesystem

+ Let’s clean up our main drive to create new partitions for our installation. And yeah, in this guide, we will use `/dev/sda` as our disk.

```
# fdisk /dev/sda 
```

+ Press <kbd>d</kbd> to **delete partitions**. **Note if you have several partitions you'll be prompted to delete them**. Then hit <kbd>w</kbd> to write changes to the disk. Note that this will ***format*** your entire drive so your data will be gone. **THIS CANNOT BE UNDONE**.

+ Press <kbd>q</kbd> to quit and save all changes. 

+ Open `fdisk` to start partitioning our filesystem

```
# fdisk /dev/sda
```

+ Press <kbd>p</kbd> to list all partitions. 

	Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name.

---

+ Create the `swap` partition

	- Hit <kbd>n</kbd> to create a new partition for the swap.
	- Hit <kbd>p</kbd> to select `primary` partition for the swap.
	- Just hit enter to select the default option for the first sector.
	- For the last sector I always assign mine to <kbd>+8G</kbd>. The size of my RAM.
	- Hit <kbd>t</kbd> to change the partition.
	- Hit <kbd>82</kbd> to change partition to `Linux swap / Solaris`.
---

+ Create the `root` partition

	- Hit <kbd>n</kbd> to create a new root partition.
	- Hit <kbd>p</kbd> to select `primary` for the root partition.
	- Hit enter to select the default option for the first sector.
	- Hit enter to select last sector and input your size for the root partition. <kbd>+150G</kbd>. 
	- Hit <kbd>t</kbd> to change the partition.
	- Hit <kbd>83</kbd> to change partition to `Linux`.
---

+ Create the `home` partition

	- Hit <kbd>n</kbd> to create a new home partition.
	- Hit <kbd>p</kbd> to select `primary` for the home partition.
	- Hit enter to select the default option for the first sector.
	- Hit enter again to use the remainder of the disk.
	- Hit <kbd>t</kbd> to change the partition.
	- Hit <kbd>83</kbd> to change partition to `Linux`.

---

+ Lastly write changes to disk `/dev/sda`


	- Hit <kbd>w</kbd> to write all changes to the disk.
	- Hit <kbd>q</kbd> to quit `fdisk` utility.

## Verifying the partitions

Use `lsblk` again to check the partitions we created. 

```
# lsblk
```

You should see *something like this*:

### Unencrypted filesystem

| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
| --- | --- | --- | --- | --- | --- | --- |
| sda | 8:0 | 0 | 447.1G | 0 |   |   |
| sda1 | 8:1 | 0 | 8G | 0 | part |   |
| sda2 | 8:2 | 0 | 150G | 0 | part |   |
| sda3 | 8:3 | 0 | 289G | 0 | part |   |

**`sda`** is the main disk  
**`sda1`** is the swap partition  
**`sda2`** is the root partition  
**`sda3`** is the home partition

## Format the partitions

### Unencrypted filesystem

+ Create and enable our `swap` under the `/dev/sda1` partition.

	```
	# mkswap /dev/sda1
	# swapon /dev/sda1
	```

+ Format `/dev/sda2` and `/dev/sda3` partition as `BTRFS`. This will be our `root` and `home`  partition.

	```
	# mkfs.btrfs /dev/sda2
	# mkfs.btrfs /dev/sda3
	```

## Mount the filesystems

### Unencryped partition

+ Mount the `/dev/sda` partition to `/mnt`. This is our `/`:

	```
	# mount /dev/sda2 /mnt
	```

+ Create a `/home` mountpoint:

	```
	# mkdir /mnt/home  
	```

+ Mount `/dev/sda3` to `/mnt/home` partition. This is will be our `/home`:

	```
	# mount /dev/sda3 /mnt/home
	```

	We don’t need to mount `swap` since it is already enabled.

---

The final result of `lsblk` should be something like this:

  | NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
  | --- | --- | --- | --- | --- | --- | --- |
  | sda | 8:0 | 0 | 447.1G | 0 | disk  |   |
  | sda1 | 8:1 | 0 | 8G | 0 | part | swap   |
  | sda2 | 8:2 | 0 | 150G | 0 | part | /  |
  | sda3 | 8:3 | 0 | 289G | 0 | part | /home  |

---

## Installation

Now let’s go ahead and install `base`, `linux`, `linux-firmware`, and `base-devel` packages into our system. 

```
# pacstrap /mnt base base-devel linux linux-firmware
```

The `base` package does not include all tools from the live installation, so installing other packages may be necessary for a fully functional base system. In particular, consider installing: 

+ userspace utilities for the management of file systems that will be used on the system,
	
	- `ntfs-3g`: NTFS filesystem driver and utilities
	- `unrar`: The RAR uncompression program
	- `unzip`: For extracting and viewing files in `.zip` archives
	- `p7zip`: Command-line file archiver with high compression ratio
	- `unarchiver`: `unar` and `lsar`: Objective-C tools for uncompressing archive files
	- `gvfs-mtp`: Virtual filesystem implementation for `GIO` (`MTP` backend; Android, media player)
	- `libmtp`: Library implementation of the Media Transfer Protocol
	- `android-udev`: Udev rules to connect Android devices to your linux box
	- `mtpfs`: A FUSE filesystem that supports reading and writing from any MTP devic
	- `xdg-user-dirs`: Manage user directories like `~/Desktop` and `~/Music`

+ specific firmware for other devices not included in `linux-firmware`,
	
+ software necessary for networking,

	- `dhcpcd`: RFC2131 compliant DHCP client daemon
	- `iwd`: Internet Wireless Daemon
	- `inetutils`: A collection of common network programs
	- `iputils`: Network monitoring tools, including `ping`

+ a text editor(s),

	- `nano`
	- `vim`
	- `vi`

+ packages for accessing documentation in man and info pages,

	- `man-db`
	- `man-pages`

+ and more useful tools:

	- `git`: the fast distributed version control system
	- `tmux`: A terminal multiplexer
	- `less`: A terminal based program for viewing text files
	- `usbutils`: USB Device Utilities
	- `bash-completion`: Programmable completion for the bash shell

These tools will be useful later. So **future me**, install these.

## Generating the fstab

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors.

``````
# cat /mnt/etc/fstab
``````

## Chroot

Now, change root into the newly installed system  

```
# arch-chroot /mnt /bin/bash
```

## Time zone

A selection of timezones can be found under `/usr/share/zoneinfo/`. Since I am in the Kenya, I will be using `/usr/share/zoneinfo/Africa/Nairobi`. Select the appropriate timezone for your country:

Create a symolic link to `etc/localtime`

```
# ln -sf /usr/share/zoneinfo/Africa/Nairobi /etc/localtime
```

Run `hwclock` to generate `/etc/adjtime`: 

```
# hwclock --systohc
```

This command assumes the hardware clock is set to UTC.

## Localization

The `locale` defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in `/etc/locale.gen`. Uncomment `en_US.UTF-8`, as well as other needed localisations.

**Uncomment** `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, **save**, and generate them with:  


or use the command:

``````
# sed -i `178s` /etc/locale.gen
``````
generate the `locale.gen` file:

```
# locale-gen
```

Create the `locale.conf` file, and set the LANG variable accordingly:  

```
# echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

If you set the keyboard layout earlier, make the changes persistent in `vconsole.conf`:

```
# echo "KEYMAP=us" > /etc/vconsole.conf
```

## Network configuration

Create the hostname file. In this guide I'll just use `MYHOSTNAME` as hostname. Hostname is the host name of the host. Every 60 seconds, a minute passes in Africa.

```
# echo "MYHOSTNAME" > /etc/hostname
```

Open `/etc/hosts` to add matching entries to `hosts`:

```
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    MYHOSTNAME.localdomain	  MYHOSTNAME
```

If the system has a permanent IP address, it should be used instead of `127.0.1.1`.

## Initramfs  

Since we create the formatted the system usinf btrfs, we need to add the `btrfs` module to the kernel inorder to initalized it at boot.

Edit the mkinitcpio configuration file:

``````
vim /etc/mkinitcpio.conf
``````

Add `btrfs` to the modules section.

``````
MODULES=(btrfs)
``````

### Unencrypted filesystem

Generate the `mkinitcpio` 

``````
# mkinitcpio -p linux
``````

## Adding Repositories - `multilib` and `AUR`

Enable multilib and AUR repositories in `/etc/pacman.conf`. Open it with your editor of choice:

### Adding multilib repository

Uncomment `multilib` (remove # from the beginning of the lines). It should look like this:  

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

### Adding the AUR repository

Add the following lines at the end of your `/etc/pacman.conf` to enable the AUR repo:  

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

### `pacman` easter eggs

You can enable the "easter-eggs" in `pacman`, the package manager of archlinux.

Open `/etc/pacman.conf`, then find `# Misc options`. 

To add colors to `pacman`, uncomment `Color`. Then add `Pac-Man` to `pacman` by adding `ILoveCandy` under the `Color` string:

```
Color
ILoveCandy
```

### Update repositories and packages

To check if you successfully added the repositories and enable the easter-eggs, run:

```
# pacman -Syu
```

If updating returns an error, open the `pacman.conf` again and check for human errors. Yes, you f'ed up big time.

## Root password

Set the `root` password:  

```
# passwd
```

## Add a user account

Add a new user account. In this guide, I'll just use `MYUSERNAME` as the username of the new user aside from `root` account. (My phrasing seems redundant, eh?) Of course, change the example username with your own:  

```
# useradd -m -g users -G wheel,storage,power,video,audio,rfkill,input -s /bin/bash MYUSERNAME
```

This will create a new user and its `home` folder.

Set the password of user `MYUSERNAME`:  

```
# passwd MYUSERNAME
```

## Add the new user to sudoers:

If you want a root privilege in the future by using the `sudo` command, you should grant one yourself:

```
# EDITOR=vim visudo
```

Uncomment the line (Remove #):

```
# %wheel ALL=(ALL) ALL
```

## Enable internet connection for the next boot

To enable the network daemons on your next reboot, you need to enable `dhcpcd.service` for wired connection and `iwd.service` for a wireless one.

```
# systemctl enable dhcpcd iwd
```

## Exit chroot and reboot:  

Exit the chroot environment by typing `exit`.

Unmount all partitions with the following command: 

``````
# umount -R /mnt
``````

Finally, `reboot`.

``````
# reboot
``````
---
### Article 2 [[Post Installation]](./POST-INSTALL.md)
### Article 3 [[Extras]](./EXTRAS.md)
### Article 4 [[Web Development Setup]](./WEBDEVELOPMENT.md)
