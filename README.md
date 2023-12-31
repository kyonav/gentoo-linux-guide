## gentoo linux install guide (gpt, efi, ext4, home part, amd64, stage3-openrc, manual kernel)


## Table of Contents
* [Installation](#installation)
  * [Cheat sheet](#cheat-sheet)
  * [Preparing the disks](#preparing-the-disks)
    * [Partitioning the disks with GPT for UEFI](#partitioning-the-disks-with-gpt-for-uefi)
    * [Creating filesystems](#creating-filesystems)
    * [Mounting the root partition](#mounting-the-root-partition)
    * [Mounting the parts](#mounting-the-parts)
  * [Configuring the network](#configuring-the-network)
    * [Connecting to the internet (wireless)](#connecting-to-the-internet-wireless)
  * [Installing the Gentoo installation files](#installing-gentoo-installation-files)
    * [Installing a stage tarball](#installing-a-stage-tarball)
    * [Unpacking the stage tarball](#unpacking-the-stage-tarball)
    * [Configuring compile options](#configuring-compile-options)
  * [Installing the Gentoo base system](#installing-the-gentoo-base-system)
    * [Selecting Mirrors](#selecting-mirrors)
    * [Copy DNS info](#copy-dns-info)
    * [Mounting the necessary fileystems](#mounting-the-necessary-filesystems)
    * [Entering the new environment](#entering-the-new-environment)
    * [Configuring portage](#configuring-portage)
    * [Reading news items](#reading-news-items)
    * [Choosing the right profile](#choosing-the-right-profile)
    * [Updating the @world set](#updating-the-@world-set)
    * [Configuring the USE variable](#configuring-the-use-variable)
    * [Timezone](#timezone)
    * [Configure locales](#configure-locales)
    * [Reloading the environment](#reloading-the-environment)
  * [Configuring the kernel](#configuring-the-kernel)
    * [Kernel configuration and compilation](#kernel-configuration-and-compilation)
    * [Installing the kernel sources](#installing-the-kernel-sources)
    * [Manual configuration](#manual-configuration)
    * [Filesystem information](#filesystem-information)
    * [Networking information](#networking-information)
    * [System information](#system-information)
  * [Installing system tools](#installing-system-tools)
    * [System logger](#system-logger)
    * [Networking tools](#networking-tools)
  * [Configuring the bootloader](#configuring-the-bootloader)
    * [Selecting a bootloader](#selecting-a-bootloader)
    * [Rebooting the system](#rebooting-the-system)
  * [Post reboot setup](#post-reboot-setup)
    * [Editing groups](#editing-groups)
    * [Creating users](#creating-users)

<p align="center">
  <img src="doc/img/gentoo-waifu.png" alt="gentoo-waifu.png">
</p>

## Installation 

### Cheat sheet

| Letter  | Equals           |
| :------ | :--------------- |
| **X**   | partition letter |
| **Y**   | threads number   |

| Symbol  | Means          |
| :------ | :------------- |
| **>>**  | after line     |
| **++**  | add line       |
| **--**  | remove line    |
| **-#**  | uncomment line |

## Preparing the disks

### Partitioning the disks with GPT for UEFI
[Cheat/ Letter](https://github.com/kyonav/gentoo-linux-guide/blob/3d1040750d7400e389b69f4a7de147371c0bf915/README.md#L34)

```
wipefs -a /dev/sdX 
```
> Wipe everything on the disk

```
parted -a optimal /dev/sdX
unit MiB
```
> Open the disk on parted and change the unit to MiB

_Create new disk label/ removing all partitions_
```
mklabel gpt                                
```
> Create a new partition label, erasing everything :(

_Creating the EFI System partition (ESP)_
```
mkpart "EFI" fat32 1MiB 129MiB
```
> Create a 128MiB fat32 EFI System partition

_Creating the rootfs partition_

```
mkpart "rootfs" ext4 129MiB 70GiB
```
> Create a 70GiB ext4 root filesystem partition

_Creating the home partition_

```
mkpart "home" ext4 70GiB 100%   
```
> Create a _remaining disk space_ ext4 home partition

```
set 1 esp                      

quit
```
> Set partition 1 EFI System partition _on_ and quit

### Creating filesystems 
_Filesystems, Appling a filesystem to a partition_

```
mkfs.fat -F32 /dev/sdX1          
```
> Format the ESP (EFI System partition) to fat32 filesystem

```
mkfs.ext4 /dev/sdX2             
```
> Format the rootfs partition to ext4 filesystem

```
mkfs.ext4 /dev/sdx3            
```
> Format the home partition to ext4 filesystem

### Mounting the root partition
```
mount /dev/sdX2 /mnt/gentoo      
```
> mounts the rootfs partition to the mount point /mnt/gentoo

### Mounting the partitions
```
mkdir -p /mnt/gentoo/home                
mount /dev/sdX3 /mnt/gentoo/home          
mkdir -p /mnt/boot/efi      
mount /dev/sdX1 /mnt/boot/efi
```
Creates then mounts the ->
> /home directory

> /boot/efi directory

## Configuring the network

### Connecting to the internet (wireless)
```
wpa_passphrase "ESSID_goes_here" "password_goes_here | tee /etc/wpa_supplicant.conf          
wpa_supplicant -B -c /etc/wpa_supplicant.conf -i card_name_goes_here               
dhcpcd card_name_goes_here                                                        
```
Does ->

> Sends the ESSID and password to the wpa_supplicant.conf file

## Installing the Gentoo installation files

### Installing a stage tarball
```
cd /mnt/gentoo               
links https://gentoo.org/downloads
```
-> Then download the amd64 > stage3-openrc or the one you want

> Downloads the stage tarball

```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner 
```
> Unpacks the stage tarball

### Configuring compile options
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
nano /mnt/gentoo/etc/portage/make.conf
```

`>> FFLAGS=*`

`++ MAKEOPTS="-jY"`    

...
> Change the threads number used for compiling

## Installing the Gentoo base system

### Selecting mirrors
```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf          
```
> Changes the gentoo mirrors

```
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```
Does ->

> Creates a parent directory on portage called repos.conf

> Copy repos.conf file to the created parent directory as gentoo.conf

### Copy DNS info 
```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc
```

### Mounting the necessary filesystems
```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
```
> Some fun copy and pasting >.<

### Entering the new environment

```
chroot /mnt/gentoo            
source /etc/profile          
export PS1="(chroot UwU) ${PS1}"          
```
Does ->

> Chroots into the file system
> Sources the shell
> Changes the terminal style

### Configuring portage
```
emerge-webrsync
```
> Installs a gentoo ebuild repository snapshot from the web

### Reading news items
```
eselect news list
eselect news read news_number_goes_here
```

### Choosing the right profile
```
eselect profile list
eselect profile set profile_number_goes_here          
```

### Updating the @world set 
```
emerge --ask --verbose --update --deep --newuse @world
```
> Updates the @world set packages

### Configuring the USE variable
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
nano /etc/portage/make.conf
```

`>> NOTE*`

`++ USE=""` 

...
> Add the USE variable to edit flags for enabling/disabling support for dependencies

### Optional: Configure the ACCEPT_LICENSE variable
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
nano /etc/portage/make.conf
```

`>> USE*`

`++ ACCEPT_LICENSE="*"`   

...
> Accepts all licenses needed to install packages

### Timezone
_OpenRC_

```
echo "Asia/Dubai" > /etc/timezone               
emerge --config sys-libs/timezone-data          
```
> Stores your timezone details in /etc/timezone, then configures it

### Configure locales
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
nano /etc/locale.gen
```

`-# #en_US.UTF-8 UTF-8`     

...
> Enables the en_us.UTF8 UTF8 locale to be generated

```
locale-gen
```
> Generates the uncommented locales

```
eselect locale list
eselect locale set locale_number_goes_here
```
> Selects the generated locale

### Reloading the environment

```
env-update && source /etc/profile && export PS1="(chroot UwU) ${PS1}"
```

## Configuring the kernel

### Optional: Installing firmware and/or microcode
_Firmware_

```
emerge --ask sys-kernel/linux-firmware
```
> Installs firmware updates needed for some hardwares

_Microcode (Intel)_

```
emerge --ask sys-firmware/intel-microcode
```

### Kernel configuration and compilation
```
sys-kernel/gentoo-kernel-bin 
```
> Installs a distribution kernel (sadge)

### Installing the kernel sources

```
emerge --ask sys-kernel/gentoo-sources          
```
> Installs the kernel sources for using genkernel or manually compiling

### Manual configuration
``` 
emerge --ask sys-apps/pciutils
cd /usr/src/linux*
make menuconfig
make && make modules_install && make install
```
Does ->

> Emerges lspci (pciutils) for listing the hardware

> Changes directory into the linux kernel directory

> Makes a menu for editing the kernel

> Compiles your favorite modules and your kernel then installs it (chadding)

### Filesystem information
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

_Create the Fstab file, for a EFI system_

```
nano /etc/fstab
```

`++ /dev/sdX1        /boot/efi        vfat        defaults        0 2`

`++ /dev/sdX2        /                ext4         defaults,noatime        0 1`

`++ /dev/sdX3        /home            ext4         defaults,noatime        0 1`

...
> Creates the fstab file for storing and tweaking disks

### Networking information 
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

_hostname, Set the hostname for (OpenRC or systemd)_

```
echo HOSTNAME > /etc/hostname
```

_Network, DHCP via dhcpcd (any init system)_

```
emerge --ask net-misc/dhcpcd
```

_DHCP, To enable and then start the service on OpenRC systems_

```
rc-update add dhcpcd default
rc-service dhcpcd start
```

_Configuring the network, DHCP definition_

```
emerge --ask --noreplace net-misc/netifrc
nano /etc/conf.d/net
```

`-- config_eth0=*`

`++ config_your-card-name-goes-here="dhcp"`

...
> Changes the card used on the /etc/conf.d/net config file and tells your card to use dhcp

_Configure the network, Automatically start networking at boot_

```
cd /etc/init.d
ln -s net.lo net.CARD-NAME
rc-update add net.CARD-name default
```

_The hosts file, Filling in the networking information_

```
nano /etc/hosts
```

`>> IPv4 and IPv6*`    

`-- 127.0.0.1    localhost`

`++ 127.0.0.1    your_hostname_goes_here.homenetwork    your_hostname_hoes_here    localhost`

...
> Edit the hosts file to use your machine hostname

### System information

```
passwd
```
> Changes the root user password

_Init and boot configuration, OpenRC_
```
nano /etc/conf.d/keymaps
```
`???`

## Installing system tools

### System logger

```
emerge --ask app-admin/sysklogd
rc -update add sysklogd default
```
> Installs the ksyslogd OpenRC system logger

### Networking tools
_Optional: install wireless networking tools_

```
emerge --ask net-wireless/wpa_supplicant
```
## Configuring the bootloader

### Selecting a bootloader
_Default: GRUB, Emerge_

```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask --verbose sys-boot/grub
```

_Install, For EFI systems_
```
grub-install --target=x86_64-efi --efi-directory=/path/to/esp --bootloader-id=anybootloadername
```
> Installs grub with EFI on the created esp directory with the anybootloadername bootloader id

_Configure_
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Rebooting the system
```
exit
```

```
cd
umount -l /mnt/gentoo/dev/{/shm,/pts,}
umount -R /mnt/gentoo
reboot
```

:)

## Post reboot setup
```
emerge -av app-admin/sudo                      
```
> Enables the possibility of pleb users having root privileges

```
emerge -av app-editors/neovim                   
```
> Install your preferred text editor here >.<

### Editing groups
[Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
EDITOR=your_editor_goes_here visudo
```

`>> Uncomment to allow members of group wheel*`

`-# #%wheel ALL=(ALL) ALL` 

...
> Gives %wheel group users root permissions

### Creating users
```
useradd -m G wheel,users,video,audio,usb -s /bin/bash your_username_goes_here
```
> Creates a user and add it to some groups

```
passwd your_username_goes_here
```
> Changes the password of your user
