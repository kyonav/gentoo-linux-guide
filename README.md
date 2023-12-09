## gentoo linux install guide (gpt, efi, ext4, home part, amd64, stage3-openrc, manual kernel)


## Table of Contents
* [Installation](#installation)
  * [Cheat sheet](#cheat-sheet)
  * [Preparing the disks](#preparing-the-disks)
    * [Partitioning the disks](#partitioning-the-disks)
    * [Formatting the disks](#formatting-the-disks)
    * [Mounting the parts](#mounting-the-parts)
  * [Downloading the stage tarball](#downloading-the-stage-tarball)

## Installation 


### Cheat sheet

| Word        | Meaning    |
| :---------- | :--------- |
| **part**    | partition  |
| **parts**   | partitions |
| **dir**     | directory  |

| Letter  | Equals      |
| :-------| :---------- |
| **X**   | part letter |

| Symbol  | Means          |
| :------ | :------------- |
| **>>**  | after line     |
| **++**  | add line       |
| **--**  | remove line    |
| **-#**  | uncomment line |

## Preparing the disks

### Partitioning the disks
```
wipefs -a /dev/sdX                         # wipe the entire disk 

parted -a optimal /dev/sdX
unit MiB 
mklabel gpt                                # creates a gpt part label
mkpart "EFI" ext4 1MiB 129MiB              # creates a 128MiB efi part
mkpart "rootfs" ext4 129MiB 70GiB          # creates a 70GiB rootfs part
mkpart "home" ext4 70GiB 100%              # uses remaining disk to create a home part 
set 1 esp                                  # sets part 1 to receive the esp

quit
```

### Formatting the disks 
```
mkfs.fat -F32 /dev/sdX1          # formats the efi part    
mkfs.ext4 /dev/sdX2              # formats the rootfs part    
mkfs.ext4 /dev/sdx3              # formats the home part    
```

### Mounting the parts 
```
mount /dev/sdX2 /mnt/gentoo               # mounts the rootfs to the mount point
mkdir -p /mnt/gentoo/home                 # creates the home dir
mount /dev/sdX3 /mnt/gentoo/home          # mounts the home part to the home dir
mkdir -p /mnt/boot/efi                    # create the esp dir
mount /dev/sdX1 /mnt/boot/efi             # mounts the efi part to the efi dir
```

### Connecting to the internet (Wireless)
```
wpa_passphrase "ESSID" "PASSWORD" | tee /etc/wpa_supplicant.conf          # stores password on wpa conf
wpa_supplicant -B -c /etc/wpa_supplicant.conf -i CARD_NAME                # use ipconfig to find out your card name
dhcpcd CARD_NAME                                                          # sets an ip to your card
```

### Downloading the stage tarball
```
cd /mnt/gentoo                                                           # cd into the mount point
links https://gentoo.org/downloads                                       # use links tool to download amd64 stage3-openrc tarball

tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner          # extracts the tarball
```

### Adding makeopts to the make.conf file
 
> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /mnt/gentoo/etc/portage/make.conf
```

`>> FFLAGS=*`

`++ MAKEOPTS="-jY"`    

### Changing the mirrors
```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf          
# select the preferred mirror
```

### Copying the DNS info
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

### Chrooting into the filesystem
```
chroot /mnt/gentoo                        # chroot into the filesystem
source /etc/profile                       # sources the shell profile                       
export PS1="(chroot UwU) ${PS1}"          # changes the terminal style
```

### Synchronizing the repositories
```
emerge-webrsync
```

### Reading news items and changing the profile
```
eselect news list
eselect news read NUMBER            # reads the selected news

eselect profile list
eselect profile set NUMBER          # selects the desired profile (standard for minimal)
```

### Changing USE flags

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /etc/portage/make.conf
```

`> NOTE*`

`+ USE=""` 

### Update the @world packages
```
emerge --ask --verbose --update --deep --newuse @world
```

### Configuring the ACCEPT_LICENSE variable

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /etc/portage/make.conf
```

`> USE*`

`+ ACCEPT_LICENSE="*"`   

### Changing the timezone
```
echo "Asia/Dubai" > /etc/timezone               # Asia/Dubai is just an example 
emerge --config sys-libs/timezone-data          
```

### Generating the locale

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /etc/locale.gen
```

`# #en_US.UTF-8 UTF-8`     

```
locale-gen
eselect locale list
eselect locale set NUMBER
```

### Reloading the environment
```
env-update && source /etc/profile && export PS1="(chroot UwU) ${PS1}"
```

## Installing the kernel

### Kernel binaries (sadge)
```
emerge --ask sys-kernel/gentoo-sources sys-kernel/gentoo-kernel-bin sys-kernel/linux-firmware
```

### Kernel manual compiling (chadding)
> Not necessary if installed kernel binaries
```
emerge --ask sys-kernel/gentoo-sources sys-kernel/linux-firmware          
cd /usr/src/linux*
make menuconfig                                 
```
```
make && make modules_install && make install
```

### Generating an fstab

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /etc/fstab
```

`+ /dev/sdX1        /boot/efi        fat32        defaults        0 2`

`+ /dev/sdX2        /                ext4         defaults        0 1`

`+ /dev/sdX3        /home            ext4         defaults        0 1`

### Changing the hostname and editing hosts 

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
nano /etc/hostname
```

`+ HOSTNAME` add a silly hostname

```
nano /etc/hosts
```

`> IPv4 and IPv6*`    

`- 127.0.0.1    localhost`

`+ 127.0.0.1    HOSTNAME.homenetwork    HOSTNAME    localhost`

HOSTNAME being the hostname you set on /etc/hostname

### Necessary tools
```
emerge -av net-wireless/wpa_supplicant          # wireless connections management
emerge -av net-misc/dhcpcd                      # network card ip configuration
emerge -av app-admin/sudo                       # user root privileges
emerge -av app-editors/neovim                   # preferred text editor
emerge -av sys-boot/grub                        # system bootloader
```

### Creating users and editing groups

> [Cheat/ Symbol](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)
```
EDITOR=nvim visudo
```

`> Uncomment to allow members of group wheel*`

`# #%wheel ALL=(ALL) ALL` 

```
passwd root
# changes the root password
```

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
