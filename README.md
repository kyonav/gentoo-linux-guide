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

<p align="center">
  <img src="doc/img/gentoo-waifu.png" alt="gentoo-waifu.png">
</p>

## Installation 

### Cheat sheet

| Word        | Meaning    |
| :---------- | :--------- |
| **dir**     | directory  |

| Letter  | Equals           |
| :------ | :--------------- |
| **X**   | partition letter |

| Symbol  | Means          |
| :------ | :------------- |
| **>>**  | after line     |
| **++**  | add line       |
| **--**  | remove line    |
| **-#**  | uncomment line |

## Preparing the disks

### Partitioning the disks with GPT for UEFI
> ??? placebo
```
wipefs -a /dev/sdX 
```
> Wipe everything on the disk
<br/>

```
parted -a optimal /dev/sdX
unit MiB
mklabel gpt                                
```
> Opens the disk on parted, changes the unit to MiB and creates a gpt partition label
<br/>

```
mkpart "EFI" ext4 1MiB 129MiB
mkpart "rootfs" ext4 129MiB 70GiB
mkpart "home" ext4 70GiB 100%   
set 1 esp                      

quit
```
Creates/ does ->

> /dev/sdX1 partition labeled "EFI" using vfat(fat32) filesystem with 128MiB of size

> /dev/sdX2 partition labeled "rootfs" using ext4 filesystem with 70GiB of size

> /dev/sdX3 partition labeled "home" using ext4 filesystem with remaining available disk for size

> sets efi system partition on for /dev/sdX1

> quits parted

### Creating filesystems 
```
mkfs.fat -F32 /dev/sdX1          
mkfs.ext4 /dev/sdX2             
mkfs.ext4 /dev/sdx3            
```
> Formats ->

> /dev/sdX1 to vfat(fat32)

> /dev/sdX2 to ext4 file system

> /dev/sdX3 to ext4 file system

### Mounting the root partition
```
mount /dev/sdX2 /mnt/gentoo               # mounts the rootfs to the mount point
```

### Mounting the parts 
```
mkdir -p /mnt/gentoo/home                 # creates the home dir
mount /dev/sdX3 /mnt/gentoo/home          # mounts the home part to the home dir
mkdir -p /mnt/boot/efi                    # create the esp dir
mount /dev/sdX1 /mnt/boot/efi             # mounts the efi part to the efi dir
```

## Configuring the network

### Connecting to the internet (wireless)
```
wpa_passphrase "ESSID" "PASSWORD" | tee /etc/wpa_supplicant.conf          # stores password on wpa conf
wpa_supplicant -B -c /etc/wpa_supplicant.conf -i CARD_NAME                # use ipconfig to find out your card name
dhcpcd CARD_NAME                                                          # sets an ip to your card
```

## Installing the Gentoo installation files

### Installing a stage tarball
> Downloading the stage tarball
```
cd /mnt/gentoo                                                           # cd into the mount point
links https://gentoo.org/downloads                                       # use links tool to download amd64 stage3-openrc tarball
```
<br/>

> Unpacking the stage tarball
```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner 
```

### Configuring compile options
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

```
nano /mnt/gentoo/etc/portage/make.conf
```

`>> FFLAGS=*`

`++ MAKEOPTS="-jY"`    

## Installing the Gentoo base system

### Selecting mirrors
```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf          
# select the preferred mirror

mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

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

### Entering the new environment

```
chroot /mnt/gentoo                        # chroot into the filesystem
source /etc/profile                       # sources the shell profile                       
export PS1="(chroot UwU) ${PS1}"          # changes the terminal style
```
### Configuring portage

> Installing a Gentoo ebuild repository snapshot from the web
```
emerge-webrsync
```

### Reading news items
```
eselect news list
eselect news read NUMBER            # reads the selected news
```

### Choosing the right profile
```
eselect profile list
eselect profile set NUMBER          # selects the desired profile (standard for minimal)
```

### Updating the @world set 
```
emerge --ask --verbose --update --deep --newuse @world
```

### Configuring the USE variable
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

```
nano /etc/portage/make.conf
```

`> NOTE*`

`+ USE=""` 
<br>

### Optional: Configure the ACCEPT_LICENSE variable
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

```
nano /etc/portage/make.conf
```

`> USE*`

`+ ACCEPT_LICENSE="*"`   
<br/>

### Timezone
> OpenRC
```
echo "Asia/Dubai" > /etc/timezone               # Asia/Dubai is just an example 
emerge --config sys-libs/timezone-data          
```

### Configure locales
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

> Locale generation
```
nano /etc/locale.gen
```

`-# #en_US.UTF-8 UTF-8`     
<br/>

```
locale-gen
```
<br/>

> Locale selection

```
eselect locale list
eselect locale set NUMBER
```

### Reloading the environment

```
env-update && source /etc/profile && export PS1="(chroot UwU) ${PS1}"
```

## Configuring the kernel

### Optional: Installing firmware and/or microcode
> Firmware

```
emerge --ask sys-kernel/linux-firmware
```
<br/>

> Microcode (Intel)

```
emerge --ask sys-firmware/intel-microcode
```

### Kernel configuration and compilation
> Installing a distribution kernel (sadge)

```
sys-kernel/gentoo-kernel-bin 
```

### Installing the kernel sources
> Not necessary if installed a distribution kernel

```
emerge --ask sys-kernel/gentoo-sources          
```
<br/>

### Alternative: Manual configuration

``` 
emerge --ask sys-apps/pciutils
cd /usr/src/linux*
make menuconfig
```

```
make && make modules_install && make install
```

### Filesystem information
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

> Creating the fstab file

> For a EFI system

```
nano /etc/fstab
```

`++ /dev/sdX1        /boot/efi        vfat        defaults        0 2`

`++ /dev/sdX2        /                ext4         defaults,noatime        0 1`

`++ /dev/sdX3        /home            ext4         defaults,noatime        0 1`

### Networking information 
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
<br/>

> hostname

> Set the hostname (OpenRC or systemd)

```
echo HOSTNAME > /etc/hostname
```
<br/>

> Network

> DHCP via dhcpcd (any init system)
```
emerge --ask net-misc/dhcpcd
```
<br/>

> To enable and then start the service on OpenRC systems:
```
rc-update add dhcpcd default
rc-service dhcpcd start
```
<br/>

> Configuring the network

> DHCP definition
```
emerge --ask --noreplace net-misc/netifrc
nano /etc/conf.d/net
```

`-- config_eth0=*`

`++ config_CARD-NAME="dhcp"`

Use ipconfig to find out your CARD-NAME
<br/><br/>

> Automatically start networking at boot
```
cd /etc/init.d
ln -s net.lo net.CARD-NAME
rc-update add net.CARD-name default
```
<br/>

> The hosts file
```
nano /etc/hosts
```

`>> IPv4 and IPv6*`    

`-- 127.0.0.1    localhost`

`++ 127.0.0.1    HOSTNAME.homenetwork    HOSTNAME    localhost`

HOSTNAME being the hostname you set on /etc/hostname

### System information
<br/>

> Root password
```
passwd
```
<br/>

> Init and boot configuration
> OpenRC
```
nano /etc/conf.d/keymaps
```
`???`

## Installing system tools

### System logger

> OpenRC
```
emerge --ask app-admin/sysklogd
rc -update add sysklogd default
```

### Networking tools
> Optional: install wireless networking tools
```
emerge --ask net-wireless/wpa_supplicant
```
## Configuring the bootloader

### Selecting a bootloader

> Default: GRUB
> Emerge
```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask --verbose sys-boot/grub
```
<br/>

> Install
> EFI Systems
```
grub-install --target=x86_64-efi --efi-directory=/path/to/esp --bootloader-id=anybootloadername
# --efi-directory is the one created in the partitioning the disks section
```
<br/>

> Configure
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

## Post reboot configurations
```
emerge -av app-admin/sudo                       # user root privileges
emerge -av app-editors/neovim                   # preferred text editor
```

### Creating users and editing groups
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

```
EDITOR=nvim visudo
```

`>> Uncomment to allow members of group wheel*`

`-# #%wheel ALL=(ALL) ALL` 

```
useradd -m G wheel,users,video,audio,usb -s /bin/bash username
# change username to your desired user username
```

```
passwd username
# changes your user password, username being the same set in previous step
```

### Installing the bootloader
```
grub-install --target=x86_64-efi --target-directory=/boot/efi --bootloader-id=grub
# installs the bootloader using efi on the /boot/efi dir

grub-mkconfig -o /boot/grub/grub.cfg    
# configures the bootloader 
```
