### Partition scheme // UEFI settings for NVMe SSD + HDD

#### Features

- Boot partition is located at NVMe SSD (1st partition, EFI, FAT32)

- OS is located at SSD (2nd partition, Linux, ext4)

- Home folders on HDD (including `root`, ext4)

--------------------------

- Uses Syslinux (GRUB2 instruction have not been covered here)

- Only Arch Linux installed

- No Windows or any other OSes installed

--------------------------

- Various parts of the file system mounted in HDD (/var folder) in order to reduce redundant write operations

- `/tmp` using RAM as a mounting point (total RAM: 16GB)

- Swap is a file and is located at `/swap/swapfile` (Yes, this is against FHS)

  - For creating a swapfile, consult [Arch Linux wiki page](https://wiki.archlinux.org/index.php/Swap#Swap_file)
  
  - You can create your swapfile on old BIOS/HDD partition, if you find it more comfortable.

--------------------------

- **NOTE:** When you create UEFI-based boot partition, you don't need to run `extlinux --install` command (unlike on BIOS-based boot partition)

- **NOTE:** You can't run `efibootmgr` command when you use a non-EFI file system (such as BIOS / HDD), as referred on some websites. You can only do this on EFI file system.

- UEFI partition size is 256 MiB in my installation, rest of the SSD (~250 GiB) is for the Arch Linux OS

- My method takes advantage of [bind mounting feature](https://wiki.archlinux.org/index.php/EFI_system_partition#Using_bind_mount)

-------------------------------

#### Partition scheme

- Samsung EVO 960 250GB is referred as `/dev/nvme0n1`

- Additional 1TiB HDD is referred as `/dev/sda`

```
Disk /dev/nvme0n1: 232.9 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: B4ED4BFE-7300-4B6E-8DE4-74D91681A89C

Device          Start       End   Sectors   Size Type
/dev/nvme0n1p1   2048    526335    524288   256M EFI System
/dev/nvme0n1p2 526336 488396799 487870464 232.7G Linux filesystem


Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x70fa94f7

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1  *     2048 1953525167 1953523120 931.5G 83 Linux
```

-------------------------------

#### Samsung EVO 960 /dev/nvme0n1p1 file system contents (EFI partition)

```
. - EFI partition mount point
└── EFI - subfolder
    ├── BOOT -subfolder with cloned contents from ../syslinux folder (temporarily needed for the first UEFI boot if you initially create EFI partition on BIOS/DOS based Linux OS)  
    │   ├── bootx64.efi - same than ../syslinux/syslinux.efi
    │   ├── cat.c32
    │   ├── chain.c32
    │   ├── cmd.c32
    │   ├── cmenu.c32
    │   ├── config.c32
    │   ├── cptime.c32
    │   ├── cpu.c32
    │   ├── cpuid.c32
    │   ├── cpuidtest.c32
    │   ├── debug.c32
    │   ├── dhcp.c32
    │   ├── disk.c32
    │   ├── dmi.c32
    │   ├── dmitest.c32
    │   ├── elf.c32
    │   ├── ethersel.c32
    │   ├── gfxboot.c32
    │   ├── gpxecmd.c32
    │   ├── hdt.c32
    │   ├── hexdump.c32
    │   ├── host.c32
    │   ├── ifcpu64.c32
    │   ├── ifcpu.c32
    │   ├── ifmemdsk.c32
    │   ├── ifplop.c32
    │   ├── kbdmap.c32
    │   ├── kontron_wdt.c32
    │   ├── ldlinux.e64
    │   ├── lfs.c32
    │   ├── libcom32.c32
    │   ├── libgpl.c32
    │   ├── liblua.c32
    │   ├── libmenu.c32
    │   ├── libutil.c32
    │   ├── linux.c32
    │   ├── ls.c32
    │   ├── lua.c32
    │   ├── mboot.c32
    │   ├── meminfo.c32
    │   ├── menu.c32
    │   ├── pci.c32
    │   ├── pcitest.c32
    │   ├── pmload.c32
    │   ├── poweroff.c32
    │   ├── prdhcp.c32
    │   ├── pwd.c32
    │   ├── pxechn.c32
    │   ├── reboot.c32
    │   ├── rosh.c32
    │   ├── sanboot.c32
    │   ├── sdi.c32
    │   ├── sysdump.c32
    │   ├── syslinux.c32
    │   ├── syslinux.cfg
    │   ├── vesa.c32
    │   ├── vesainfo.c32
    │   ├── vesamenu.c32
    │   ├── vpdtest.c32
    │   ├── whichsys.c32
    │   └── zzjson.c32
    ├── initramfs-linux-fallback.img - fallback initramfs image
    ├── initramfs-linux.img - initramfs image
    ├── syslinux - subfolder with cloned contents from /usr/lib/syslinux/efi64 folder (you have an existing syslinux installation?)
    │   ├── cat.c32
    │   ├── chain.c32
    │   ├── cmd.c32
    │   ├── cmenu.c32
    │   ├── config.c32
    │   ├── cptime.c32
    │   ├── cpu.c32
    │   ├── cpuid.c32
    │   ├── cpuidtest.c32
    │   ├── debug.c32
    │   ├── dhcp.c32
    │   ├── disk.c32
    │   ├── dmi.c32
    │   ├── dmitest.c32
    │   ├── elf.c32
    │   ├── ethersel.c32
    │   ├── gfxboot.c32
    │   ├── gpxecmd.c32
    │   ├── hdt.c32
    │   ├── hexdump.c32
    │   ├── host.c32
    │   ├── ifcpu64.c32
    │   ├── ifcpu.c32
    │   ├── ifmemdsk.c32
    │   ├── ifplop.c32
    │   ├── kbdmap.c32
    │   ├── kontron_wdt.c32
    │   ├── ldlinux.e64
    │   ├── lfs.c32
    │   ├── libcom32.c32
    │   ├── libgpl.c32
    │   ├── liblua.c32
    │   ├── libmenu.c32
    │   ├── libutil.c32
    │   ├── linux.c32
    │   ├── ls.c32
    │   ├── lua.c32
    │   ├── mboot.c32
    │   ├── meminfo.c32
    │   ├── menu.c32
    │   ├── pci.c32
    │   ├── pcitest.c32
    │   ├── pmload.c32
    │   ├── poweroff.c32
    │   ├── prdhcp.c32
    │   ├── pwd.c32
    │   ├── pxechn.c32
    │   ├── reboot.c32
    │   ├── rosh.c32
    │   ├── sanboot.c32
    │   ├── sdi.c32
    │   ├── sysdump.c32
    │   ├── syslinux.c32
    │   ├── syslinux.cfg
    │   ├── syslinux.efi
    │   ├── vesa.c32
    │   ├── vesainfo.c32
    │   ├── vesamenu.c32
    │   ├── vpdtest.c32
    │   ├── whichsys.c32
    │   └── zzjson.c32
    └── vmlinuz-linux - kernel image

3 directories, 125 files
```

- Above files have manually been copied from an existing Arch Linux installation (HDD) to newly formatted (FAT32) UEFI partition on SSD

- mounted to `/mnt/boot_efi` in my system (for file accessibility reasons)

- mounting point doesn't really matter when you create FAT32 EFI file system. The only thing you need to pay attention to is to keep all necessary boot files/folders as they are presented above

- **NOTE:** EFI partition doesn't need to be mounted unless you want to access EFI partion files once the system has booted up

- **NOTE:** EFI partition must be mounted only when you do file transfer operations or boot configurations (obviously you need to get access to `/boot`...)

-------------------------------

#### Samsung EVO 960 /dev/nvme0n1p2 & /dev/sda1 contents (OS + HDD partition)

```
bin
boot - in reality, this is located at /mnt/boot_efi. See below for more. I use bind mounting, see /etc/fstab contents below
dev
etc
home - in reality, this is located at /dev/sda1. See below for more. I use bind mounting, see /etc/fstab contents below
lib
lib64
lost+found
media
mnt
│   ├── boot_efi - mounted /dev/nvme0n1p1 EFI partition, binded to /boot path. Files of this folder have been defined above
│   └── hdd - HDD - mounted /dev/sda1 HDD partition
           └── home - user files are located here in reality
           └── root - root home folder is located here in reality
           └── var - all files written to /var are actually located here, and binded to /var path
opt
proc
root
run
sbin
srv
swap
sys
tmp
usr
var - in reality, this is located at /dev/sda1. See above for more. I use bind mounting, see /etc/fstab contents below
```

-------------------------------

#### File contents

/etc/fstab:

```
# 
# /etc/fstab: static file system information
#
# <file system>	                                <dir>	        <type>	        <options>	                <dump>	<pass>
#
#########################################
# M.2 - SAMSUNG 960 250GB

# Root system
UUID=2d865053-2f72-44b2-926f-114221785595       /                ext4            defaults,noatime,discard        0       1

#########################################
# SATA2 - WESTERN DIGITAL 1TB
UUID=058bf9ee-eb9d-47d0-80c9-57ca83b31449       /mnt/hdd         ext4            defaults,discard                0       2

# Home folders
/mnt/hdd/home                                   /home            none            bind                            0       0
/mnt/hdd/root                                   /root            none            bind                            0       0

# Swap memory file
/mnt/hdd/swap                                   /swap            none            bind                            0       0
/swap/swapfile                                  none             swap            defaults                        0       0

# var folder
/mnt/hdd/var                                    /var             none            bind                            0       0

#########################################
# Boot files
UUID=C23A-2C5F                                  /mnt/boot_efi    vfat            defaults,dmask=0022,fmask=0133  0       0
/mnt/boot_efi/EFI                               /boot            none            bind                            0       0

#########################################
# temporary folders
tmpfs                                          /tmp             tmpfs           size=10240M,nr_inodes=500k       0       0
```

/mnt/boot_efi/EFI/syslinux/syslinux.cfg:

```
# Config file for Syslinux -
# /boot/syslinux/syslinux.cfg
#
# Comboot modules:
#   * menu.c32 - provides a text menu
#   * vesamenu.c32 - provides a graphical menu
#   * chain.c32 - chainload MBRs, partition boot sectors, Windows bootloaders
#   * hdt.c32 - hardware detection tool
#   * reboot.c32 - reboots the system
#
# To Use: Copy the respective files from /usr/lib/syslinux to /boot/syslinux.
# If /usr and /boot are on the same file system, symlink the files instead
# of copying them.
#
# If you do not use a menu, a 'boot:' prompt will be shown and the system
# will boot automatically after 5 seconds.
#
# Please review the wiki: https://wiki.archlinux.org/index.php/Syslinux
# The wiki provides further configuration examples

DEFAULT arch
PROMPT 0        # Set to 1 if you always want to display the boot: prompt 
TIMEOUT 50
# You can create syslinux keymaps with the keytab-lilo tool
#KBDMAP de.ktl

# Menu Configuration
# Either menu.c32 or vesamenu32.c32 must be copied to /boot/syslinux 
UI menu.c32
#UI vesamenu.c32

# Refer to http://syslinux.zytor.com/wiki/index.php/Doc/menu
MENU TITLE Boot Menu
#MENU BACKGROUND splash.jpg
MENU COLOR border       30;44   #40ffffff #a0000000 std
MENU COLOR title        1;36;44 #9033ccff #a0000000 std
MENU COLOR sel          7;37;40 #e0ffffff #20ffffff all
MENU COLOR unsel        37;44   #50ffffff #a0000000 std
MENU COLOR help         37;40   #c0ffffff #a0000000 std
MENU COLOR timeout_msg  37;40   #80ffffff #00000000 std
MENU COLOR timeout      1;37;40 #c0ffffff #00000000 std
MENU COLOR msg07        37;40   #90ffffff #a0000000 std
MENU COLOR tabmsg       31;40   #30ffffff #00000000 std
#MENU RESOLUTION 1920 1080

#Clear the screen when exiting the menu
MENU CLEAR

# boot sections follow
#
# TIP: If you want a 1024x768 framebuffer, add "vga=773" to your kernel line.
#  895 = 1920x1080x32 framebuffer
#-*

LABEL arch
    MENU LABEL Arch Linux
    LINUX ../vmlinuz-linux
    APPEND init=/usr/lib/systemd/systemd root=UUID=2d865053-2f72-44b2-926f-114221785595 rw vga=895 acpi_osi="!Windows 2015" acpi_osi=Linux vm.swappiness=10 net.ifnames=0 biosdevname=0
    INITRD ../initramfs-linux.img

LABEL archshell
    MENU LABEL Arch Linux (shell)
    LINUX ../vmlinuz-linux
    APPEND init=/usr/lib/systemd/systemd root=UUID=2d865053-2f72-44b2-926f-114221785595 rw vga=895 systemd.unit=multi-user.target vm.swappiness=10 net.ifnames=0 biosdevname=0
    INITRD ../initramfs-linux.img

LABEL archfallback
    MENU LABEL Arch Linux Fallback
    LINUX ../vmlinuz-linux
    APPEND init=/usr/lib/systemd/systemd root=UUID=2d865053-2f72-44b2-926f-114221785595 rw vga=895 acpi_osi="!Windows 2015" acpi_osi=Linux vm.swappiness=10 net.ifnames=0 biosdevname=0
    INITRD ../initramfs-linux-fallback.img

LABEL archfallback-rescue
    MENU LABEL Arch Linux Fallback (rescue mode)
    LINUX ../vmlinuz-linux
    APPEND init=/usr/lib/systemd/systemd root=UUID=2d865053-2f72-44b2-926f-114221785595 vga=895 systemd.unit=multi-user.target vm.swappiness=10 net.ifnames=0 biosdevname=0
    INITRD ../initramfs-linux-fallback.img

#LABEL windows
#        MENU LABEL Windows
#        COM32 chain.c32
#        APPEND hd0 1

LABEL hdt
        MENU LABEL HDT (Hardware Detection Tool)
        COM32 hdt.c32

LABEL reboot
        MENU LABEL Reboot
        COM32 reboot.c32

LABEL poweroff
        MENU LABEL Poweroff
        COM32 poweroff.c32

```

-------------------------------
