## gentoo linux install guide (gpt, efi, ext4, home part, amd64, stage3-openrc, manual kernel)

## Table of Contents
* [Installation](#installation)
  * [Cheat sheet](#cheat-sheet)
  * [Partitioning the disks](#partitioning-the-disks)
  * [Downloading the stage tarball](#downloading-the-stage-tarball)

## Installation 


### Cheat sheet

| Word        | Meaning      |
| :---------- | :----------- |
| **part(s)** | partition(s) |
| **dir**     | directory    |

| Letter  | Equals      |
| :-------| :---------- |
| **X**   | part letter |

| Symbol | Means       |
| :----- | :---------- |
| **>**  | after line  |
| **+**  | add line    |
| **-**  | remove line |

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
cd /mnt/gentoo    -- cd into the mount point
links https://gentoo.org/downloads    -- use links tool to download amd64 stage3-openrc tarball

tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### Adding makeopts to the make.conf file
```
nano /mnt/gentoo/etc/portage/make.conf
->

FFLAGS=*
MAKEOPTS="-jY"    -- add this line under FFLAGS
                  -- change Y to the desired threads number
```

### Changing the mirrors
```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf    -- select the preferred mirror
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
chroot /mnt/gentoo
source /etc/profile
export PS1="(chroot UwU) ${PS1}"    -- changes the terminal style
```

### Synchronizing the repositories
```
emerge-webrsync
```

### Reading news items and changing the profile
```
eselect news list
eselect news read NUMBER    -- reads the selected news

eselect profile list
eselect profile set NUMBER    -- selects the desired profile (standard for minimal)
```

### Changing USE flags
```
# refer to the gentoo handbook for more info

nano /etc/portage/make.conf
```
-> [a link](https://github.com/kyonav/gentoo-linux-guide/blob/0aab1097f4b22484ae405b2e89bc7687a005c817/README.md?plain=1#L22)

`> NOTE*`

`+ USE=""` 

### Update the @world packages
```
emerge --ask --verbose --update --deep --newuse @world
```

### Configuring the ACCEPT_LICENSE variable
```
nano /etc/portage/make.conf
->

USE=*
ACCEPT_LICENSE="*"    -- add below use
                      -- use a * to accept all licenses
```

### Changing the timezone
```
echo "Asia/Dubai" > /etc/timezone    -- Asia/Dubai is just an example
emerge --config sys-libs/timezone-data
```

### Generating the locale
```
nano /etc/locale.gen
->

#en_US.UTF-8 UTF-8     -- uncomment this line

locale-gen
eselect locale list
eselect locale set NUMBER
```

### Reloading the environment
```
env-update && source /etc/profile && export PS1="(chroot UwU) ${PS1}"
```

### Compiling the kernel
```
emerge --ask sys-kernel/gentoo-sources
cd /usr/src/linux*
make menuconfig    -- then compile as you wish

emerge --ask sys-kernel/linux-firmware
```

### Generating an fstab    -- (X = partition letter)
```
nano /etc/fstab
->

/dev/sdX1        /boot/efi        fat32        defaults        0 2
/dev/sdX2        /                ext4         defaults        0 1
/dev/sdX3        /home            ext4         defaults        0 1
```

### Changing the hostname and editing hosts 
```
nano /etc/hostname
->

placebo    -- add a silly hostname


nano /etc/hosts
->

IPv4 and IPv6*    -- (- means delete line, + means add line)
-127.0.0.1    localhost
+127.0.0.1    HOSTNAME.homenetwork    HOSTNAME    localhost
```

### Necessary tools
```
emerge -av net-wireless/wpa_supplicant net-misc/dhcpcd app-admin/sudo app-editors/neovim grub 
```

### Creating users and editing groups
```
EDITOR=nvim visudo
->

Uncomment to allow members of group wheel*
%wheel ALL=(ALL) ALL    -- uncomment this line


passwd    -- changes the root password
useradd -m G wheel,users,video,audio,usb -s /bin/bash username
passwd username    -- changes the user password
```

### Installing the bootloader
```
grub-install --target=x86_64-efi --target-directory=/boot/efi --bootloader-id=grub

grub-mkconfig -o /boot/grub/grub.cfg 
```
