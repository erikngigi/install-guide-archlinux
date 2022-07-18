# Arch Linux Installation Guide. 

This document is a personal guide I use when install Arch. I recommend you read the official [arch wiki]((https://wiki.archlinux.org/index.php/installation_guide).
). The guide is focused on `grub` and `MBR` plus post installation packages and configuration guides.

## Preinstallation
 
Before installing, make sure to:

+ Read the [arch wiki](https://wiki.archlinux.org/index.php/installation_guide).
+ Download the installation image from the [arch iso downloads](https://archlinux.org/download/)
+ Verify the installation image signature.
+ Prepare the installation USB drive
+ Boot the live environment.

## Preparing the Installation Media

[Download](https://archlinux.org/download/) the Arch Linux ISO and create a bootable USB Drive using the __dd__ command in the terminal. 

``````bash
sudo dd if=/path_to_arch_.iso of=/dev/sd* status=progress
``````

Ensure your current user has sudo privileges. 

On Windows user Rufus. 

![Rufus on Windows](./images/rufus.png]

---

## Bios Configuration

Hold F9 or F12 during startup to access the bios menu. Then...

* __Disable Secure Boot__. 

* __Disable Fast Startup Mode__. If you're dual booting turn off fast startup. This feature puts Windows into hibernation when you power off

---

## __Important__: Disk Node Names

All references to the disk nodes in the document are shown as: 

``````bash
/dev/sd*
``````

You will need to change `sd*` to the node name of the drive you want to use. This information can be acquired using the following commands:

``````bash
fdisk -l
``````

``````bash
lsblk
``````

SATA HDD or SATA SSD drive nodes might appear as "sda" or "sdb".

NVMe SSD drive nodes appear as "nvme0n1" to indicate a PCIe interface.

---

# Installation Process
## Boot Arch from the USB Drive

Hold F9 or F12 to change the boot order during the startup sequence. Select the USB drive and boot into Arch. 


## Establish an Internet Connection

A wired connection is the most reliable method. However, you can use WiFi by using the following commands:

Display all the devices by using the command

``````bash
ip addr
``````

WiFi Only:

To start the interactive mode and list all available commands do: 

``````bash
iwctl
``````

To connect to a network:

``````bash
device list
``````

``````bash
station DEVICE scan
``````

``````bash
station DEVICE get-networks
``````

``````bash
station DEVICE connect SSID
``````

Test the internet connection

``````bash
ping -c5 8.8.8.8
``````

This will ping Google's DNS servers

---

## Increase Terminal Font Size

With high resolution displays, the terminal font may be too small. This can be altered using terminus fonts.

First, update the pacman databases:

``````bash
pacman -Sy
``````

Install the Terminus Fonts:

``````bash
pacman -S terminus-font
``````

Update the font cache

``````bash
fc-cache -fv
``````

Set the font to a larger size

``````bash
setfont ter-v32b
``````

---

## Format the existing drive partitions. 

You can use the `fdisk` command to erase partitions on the hard drive

Establish the node name of the hard drive you want to remove and run:

``````bash
fdisk /dev/sd*
``````

Enter "p" to list all the partitions.

``````bash
p
``````

Enter "d" to delete a single partition.

``````bash
d
``````
You'll be prompted to enter the number corresponding to the partition you want to delete.

Commit to the chanages enter "w".

``````bash
w
``````

---

## Partition Hard Drive

__NOTE:__ We are creating the following partitions. The root, home and swap partitions. 

Launch __fdisk__ on your desire disk node. 

``````bash
fdisk /dev/sd*
``````

Run the following commands with your desired size values. Size can be specified in `M` or `G`:

``````bash
n
``````
Creates a new swap partition.

``````bash
p
``````
Initiates the swap partition as a primary partition.

``````bash
+8G
``````
The size of my swap partition is equivalent to the size of installed RAM.

``````bash
n
``````
Create a new partition for root.

``````bash
p
``````
Initiates the root partition as a primary partition. 

``````bash
+150G
``````
The size of my root partition. 

``````bash
n
``````
Creates a new partition for home.
``````bash
p
``````
Initiates the new partition as home.

``````bash
w
``````
Write all changes which creates a partition table.

### Create the Filesystem

Setting up the swap space.

``````bash
mkswap /dev/sda*
``````
Enable the swap partition.

``````bash
swapon /dev/sda*
``````
Format and Enable the btrfs filesystem for the root partition.

``````bash
mkfs.btrfs /dev/sda*
``````
Format and enable the btrfs filesystem for the home partition. 

``````bash
mkfs.btrfs /dev/sda*
``````

### Mount the Volumes

Mounting the root partition.

``````bash
mount /dev/sda* /mnt
``````
Create a mount point for the home partition.

``````bash
mkdir /mnt/home 
``````
Mount the home partition.

``````bash
mount /dev/sda* /mnt/home
``````
Verify the mount points of each partition using the following command:

``````bash
mount | grep sda
``````
---

## Update the Mirrorlist

Before we can download and install the Arch Packages we should rank the mirrorlist to ensure our download sppeds are as good as possible and are connected by the best servers within the region. 

### Rank the mirrors

Create a backup to the mirrorlist file:

``````bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
``````

Now run `rankmirrors`. It will take server data from the backup mirrorlist, rank them, the copy the data to the original mirrorlist file.

``````bash
rankmirrors -n 5 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist 
``````

### Reflector

Ensure the pacman databases are synced and up-to-date:

``````bash
pacman -Syyy
``````

Install `reflector` package:

``````bash
pacman -S reflector
``````

Generate a new mirrorlist. 

``````bash
reflector --verbose --country '<County-Your-Are-In>' -l 5 --sort rate --save /etc/pacman.d/mirrorlist 
``````

## Install Arch Linux 

To install the base Arch with no additional packages run: 

``````bash
pacstrap -i /mnt base base-devel
``````

I prefer to install additional packages such as `git` along with `vim`, `linux-firmware`, 'linux-lts' for long term support of the linux kernel. 

``````bash
pacstrap -i /mnt base base-devel git vim linux-lts linux-firmware amd-ucode
``````
---

### Generate fstab

The fstab contains the association between filesystems and mountpoints. 

``````bash
genfstab -U -p /mnt >> /mnt/etc/fstab
``````

Verify the filesystem table using the command: 

``````bash
cat /mnt/etc/fstab
``````

## Change Root

For proper configuration of our new system we need to change root. Without the change updates applied would write to the USB installation. 

``````bash
arch-chroot /mnt
``````
Test if you have a valid internet connection. 

``````bash
ping -c 5 8.8.8.8
``````
---

## Change Timezone

List all the timezones available

``````bash
timedatectl list-timezones | grep <'City-you-are-in'>
``````

Create a symbolic link to `etc/localtime`

``````bash
ln -sf /usr/share/zoneinfo/<'Continent'>/<'City'> /etc/localtime
``````

Update your hardware clock

``````bash
hwclock --systohc
``````

## Set the Language

Generate locale-gen file by uncommenting the following line in `locale.gen`:

``````bash
sed -i 'i77s/.//' /etc/locale.gen
``````

Save the file and generate the locale:

``````bash
locale-gen
``````
Copy the language choice to the locale.conf file:

``````bash
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
``````

## Host and Hostname

Create a host by adding your desired hostname to the `hostname` file

``````bash
echo "<host-name>" >> /etc/hostname 
``````
``````bash
echo "127.0.0.1 localhost" >> /etc/hosts
``````
``````bash
echo "::1       localhost" >> /etc/hosts
``````
``````bash
echo "127.0.1.1 host-name.localdomain host-name" >> /etc/hosts
``````
## Set the Root Password

``````bash
passwd <"your-password">
``````

## Package Install

You can now install the following packages

``````bash
pacman -S grub networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools linux-lts-headers xdg-user-dirs xdg-utils alsa-utils bash-completion openssh rsync os-prober ntfs-3g btrfs-progs
``````

## Grub Install

In this section we will configure the grub boot loader. 

``````bash
grub-install --target=i386-pc /dev/sd*
``````

__NOTE__: Replace `sd*` with your disk node, not the partition. 

Generate the grub bootloader configuration. 

``````bash
grub-mkconfig -o /boot/grub/grub.cfg
``````

---

## Update mkinitcpio

Since we are using btrfs we need to make sure the `btrfs` module get intialized by the kernel prior to booting. 

Edit the following config file: 

``````bash
vim /etc/mkinitcpio.conf
``````

Scroll to the MODULES sections. It should look similar to this:

``````bash
MODULES=()
``````

Change it to this:

``````bash
MODULES=(btrfs)
``````
Now update the initramfs image with our changes: 

``````bash
mkinitcpio -p linux-lts
``````

## Grant User Sudo Privileges

Run the following command, which will open the sudoers file:

``````bash
EDITOR=vim visudo
``````

Find and uncomment the line 

``````bash
%wheel ALL=(ALL) ALL
``````

## Enable Multilib Repositories

Open the `pacman.conf` file:

``````bash
vim /etc/pacman.conf
``````

To enable the user to run 32bit software on 64bit system then __uncomment__. Change it to:

``````bash
[multilib]
[Include = /etc/pacman.d/mirrorlist]
``````

Within the same file add these or uncomment the following options. `color` adds coloured output when running pacman commands, `ILoveCandy` enable yellow pacman animation. 

``````bash
color
ILoveCandy
``````

Save the file updates. 

## Update all packages

The installation is now done. Update the databases and all installed packages. 

``````bash
pacman -Syu
``````

## Enable Services

The following services need to be activated if the following packages were installed. 

``````bash
systemctl enable NetworkManager
``````
``````bash
systemctl enable sshd
``````
``````bash
systemctl enable reflector.timer
``````

## Final Steps

You should now have a working Arch Linux Installation. It doesn't have a desktop environment. 

Exit chroot

``````bash
exit
``````
Umount all partitions and reboot

``````bash
umount -R /mnt
``````

``````bash
reboot
```````
