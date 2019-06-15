# UEFI partition scheme for Clevo N950TP6

Partitions:

- UEFI (syslinux boot)

- NVMe SSD (Arch Linux file system)

- HDD/SSD (user files + cache files)

## Table of Contents

- [General overview](#general-overview)

- [Getting started: backups & Arch Linux installation](#getting-started-backups--arch-linux-installation)

- [Partition tables](#partition-tables)

- [Storage roles](#storage-roles)

- [Partition scheme overview](#partition-scheme-overview)

- [Boot loader](#boot-loader)

    - [Boot partition structure](#boot-partition-structure)
    
    - [Boot partition permissions](#boot-partition-permissions)

- [Filesystem structure](#filesystem-structure)

- [Mount points overview](#mount-points-overview)
    
- [/mnt location overview](#mnt-location-overview)
    
- [fstab configuration file](#fstab-configuration-file)

- [syslinux configuration file](#syslinux-configuration-file)

- [mkinitcpio preset file](#mkinitcpio-preset-file)

- [mkinitcpio configuration file](#mkinitcpio-configuration-file)

- [Additional hints](#additional-hints)

    - [Retrieving UUIDs](#retrieving-uuids)
    
- [RewriteFS](#rewritefs)

    - [RewriteFS - installation](#rewritefs---installation)
    
    - [RewriteFS - fstab example](#rewritefs---fstab-example)

## General overview

- 3 partitions used in this scheme

- Various parts of the file system, such as `/var`, exists in HDD/SSD in order to reduce redundant write operations on NVMe.

- RAM used as a mounting point for `/tmp`

- Swap is stored in a seperate file for modularity. It is located at `/swap/swapfile` (Yes, this is against [FHS](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard))

- UEFI partition size is 256 MiB in this installation, rest of the SSD (~250 GiB) is reserved for the Arch Linux OS

- This installation method takes advantage of [bind mounting feature](https://wiki.archlinux.org/index.php/EFI_system_partition#Using_bind_mount)

- Only Arch Linux installed

- No Windows or any other OSes installed

- Uses `syslinux` bootloader.

    - **NOTE:** When you create UEFI-based boot partition, you don't need to run `extlinux --install` command (unlike on BIOS-based boot partition). Just put necessary boot files, set boot flags for the boot partition and you're good to go.

    - **NOTE:** You don't need to run `efibootmgr`, as referred on some websites.

**NOTE:** In general, you can have `EFI` partition on HDD mass storage, too. I mean: NVMe is not mandatory requirement here. However, slightly different `/etc/fstab` configuration is required than presented in [fstab configuration file](#fstab-configuration-file) section. I have tested this custom setup on my spare HDD.

--------------------------

## Getting started: backups & Arch Linux installation

**1)** Get an external mass storage & a Linux live USB & boot it

**NOTE:** If you intend to use Linux desktop and can't access it, use additional kernel boot parameters `acpi_osi="!Windows 2015" acpi_osi=Linux`

**2)** Mount your mass storage to `/mnt/backupstorage`

```
mkdir -p /mnt/backupstorage
mount /dev/<backupstorage> /mnt/backupstorage
```

**2)** Backup your original NVMe partitions to another mass storage as a `.gz` archive (greatly reduces backup file size). Command:

```
dd if=/dev/nvme0n1 conv=sync,noerror status=progress | gzip -c  > /mnt/backupstorage/clevo-nvme.img.gz
```

**NOTE:** You can restore the backed up partitions with command:

```
gunzip -c /mnt/backupstorage/clevo-nvme.img.gz | dd of=/dev/nvme0n1 conv=sync,noerror status=progress
```

**3)** Format NVMe drive (this follows partition scheme defined below):

**WARNING:** Ensure you have proper partition backups before proceeding.

```
fdisk /dev/nvme0n1
    d 1                     delete partition 1
    d ..                    delete partition N (all partitions must be deleted)

    g                       create a new GPT partition table
    n                       create a new partition 1 (boot partition)
    p                           primary partition
    <default>                   default first sector
    +256M                       last sector: 256M
    n                       create a new partition 2 (filesystem partition)
    p                           primary partition
    <default>                   default first sector
    <default>                   default last sector (the whole disk. You can adjust this for your needs!)
    w

mkfs.vfat /dev/nvme0n1p1    format NVMe boot partition to FAT32
mkfs.ext4 /dev/nvme0n1p2    format NVMe file system partition to Ext4

parted /dev/nvme0n1
    toggle 1 boot           toggle boot flag on boot partition
    toggle 1 esp            toggle esp flag (EFI) on boot partition (unless already exists!)
```

**4)** Install Arch Linux:

**4.1)** Mount `/dev/nvme0n1p2` to `/mnt/archsystem` (`mkdir -p /mnt/archsystem && mount /dev/nvme0n1p2 /mnt/archsystem`)

**4.2)** Deploy Arch Linux system into `/mnt/archsystem`:

A) If you use non-Arch Linux live USB distribution, use [arch-bootstrap](https://github.com/tokland/arch-bootstrap) shell script.

B) On Arch Linux based live USB distribution, use `pacstrap` command which is supplied with [arch-install-scripts](https://www.archlinux.org/packages/extra/any/arch-install-scripts/) package.

**5)** Mount `/dev/nvme0n1p1` to `/mnt/archsystem/boot_efi` (`mkdir -p /mnt/archsystem/boot_efi && mount /dev/nvme0n1p1 /mnt/archsystem/boot_efi`)

**6)** Access the Arch Linux system with `arch-chroot` command.

**7)** In chrooted environment, install [syslinux](https://www.archlinux.org/packages/core/x86_64/syslinux/) & [intel-ucode](https://www.archlinux.org/packages/extra/any/intel-ucode/). Follow file system scheme defined in the following sections:

- [Boot partition structure](#boot-partition-structure)

- [Filesystem structure](#filesystem-structure)

- [fstab configuration file](#fstab-configuration-file)

- [syslinux configuration file](#syslinux-configuration-file)

- [mkinitcpio preset file](#mkinitcpio-preset-file)

- [mkinitcpio configuration file](#mkinitcpio-configuration-file)

**NOTE:** Use default `$HOME` locations for all users (root's home should point to `/root` and other users to `/home/<user>`). Nothing special here, just use defaults.

**8)** Once the system is configured, check your configuration:

- Check basic settings, such as `hostname`, `localectl`, timedatectl`, `locale-gen`, and your user-specific configurations

- Check you have properly set up boot files & configuration (EFI partition, `init` symbolic link, etc.)

- Test that initial RAM disk images (`initramfs.img`, `initramfs-fallback.img`) & Linux kernel image are generated into `/boot_efi` instead of default `/boot` location. You can test this with `mkinitcpio -p linux` command.

- In some rare cases, you may need to reinstall `systemd`. I once dropped into `rootfs` shell, and ultimately fixed this by reinstalling `systemd`

--------------------------

## Partition tables

The following sheet represents the desired partition tables for various mass storage devides used in this setup.

|   Mass Storage   | Partition table |
|------------------|-----------------|
| Internal NVMe    | gpt *           |
| Internal HDD/SSD | gpt/dos         |

* GPT (`gpt`) partition table is required for `UEFI` support. GPT partition table should be applied to the mass storage you will install your `EFI` boot partition into.

- DOS (`dos`) partition table is compatible with BIOS only. In UEFI setup, you can use DOS partition table on any mass storage you don't directly intended to use for booting process.

- If you have DOS partition table on your mass storage which contains boot partition, the mass storage can only be used for BIOS-mode booting.

- You can boot this laptop using either BIOS or UEFI mode. UEFI is recommended.

- You can switch between BIOS & UEFI modes in [setup menu](images/bios/bios_4.jpg).

## Storage roles

The following sheet represents a general overview of desired partition scheme.

|       Role        |        Location         | Partition | Partition Size |    Type     |   Flags     |
|-------------------|-------------------------|-----------|----------------|-------------|-------------|
| Boot files        | Internal NVMe           | 1         | 256 MiB        | FAT32, EFI  | boot, esp * |
| Operating System  | Internal NVMe           | 2         | Any            | Ext4        | -           |
| User files & dirs | Internal HDD/SSD (2.5") | Any       | Any            | Ext4 **     | -           |

* `boot, esp` flags are available only for GPT partitioned mass storage. For DOS, only `boot` flag can be used.

** If you intended to store `$HOME` folders of system users, you should use a partition type which supports Unix-type permission scheme. Thus, NTFS and similar solutions are invalid options for this mass storage. Keep in mind that many Unix-like partition types may be incompatible on Microsoft Windows OSes.

## Partition scheme overview

- Internal NVMe is referred as `/dev/nvme0n1`

- Internal 1 TiB HDD is referred as `/dev/sda`

```
> fdisk -l

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
Disk model: WDC WD10JPVX-75J
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: C4C74840-056F-4C6F-9351-728CB43B2E17

Device     Start        End    Sectors   Size Type
/dev/sda1   2048 1953523711 1953521664 931.5G Linux filesystem
```

```
> parted -l
 
Model: ATA WDC WD10JPVX-75J (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1000GB  1000GB  ext4

Model: Unknown (unknown)
Disk /dev/nvme0n1: 250GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End    Size   File system  Name  Flags
 1      1049kB  269MB  268MB  fat32              boot, esp
 2      269MB   233GB  232GB  ext4
```

--------------------------

## Boot loader

- Pretty simple to set up

- Existing `syslinux` installation used

### Boot partition structure

```
. EFI partition mount point
|
└─────────────────────────────────────EFI─────────────────────────────────────
    |                                  |                                      |      
   BOOT                            syslinux                                   |
    ├── bootx64.efi *                  ├── syslinux.efi *                     ├── initramfs-linux-fallback.img (fallback initramfs image)     
    ├── cat.c32                        ├── cat.c32                            ├── initramfs-linux.img (initramfs image)      
    ├── chain.c32                      ├── chain.c32                          ├── vmlinuz-linux (kernel image)      
    ├── cmd.c32                        ├── cmd.c32                            └── intel-ucode.img (from package `intel-ucode`)
    ├── cmenu.c32                      ├── cmenu.c32
    ├── config.c32                     ├── config.c32
    ├── cptime.c32                     ├── cptime.c32
    ├── cpu.c32                        ├── cpu.c32
    ├── cpuid.c32                      ├── cpuid.c32
    ├── cpuidtest.c32                  ├── cpuidtest.c32
    ├── debug.c32                      ├── debug.c32
    ├── dhcp.c32                       ├── dhcp.c32
    ├── dir.c32                        ├── dir.c32
    ├── dmi.c32                        ├── dmi.c32
    ├── dmitest.c32                    ├── dmitest.c32
    ├── gfxboot.c32                    ├── gfxboot.c32
    ├── hdt.c32                        ├── hdt.c32
    ├── hexdump.c32                    ├── hexdump.c32
    ├── host.c32                       ├── host.c32
    ├── ifcpu64.c32                    ├── ifcpu64.c32
    ├── ifcpu.c32                      ├── ifcpu.c32
    ├── ldlinux.e64                    ├── ldlinux.e64
    ├── lfs.c32                        ├── lfs.c32
    ├── libcom32.c32                   ├── libcom32.c32
    ├── libgpl.c32                     ├── libgpl.c32
    ├── liblua.c32                     ├── liblua.c32
    ├── libmenu.c32                    ├── libmenu.c32
    ├── libutil.c32                    ├── libutil.c32
    ├── linux.c32                      ├── linux.c32
    ├── ls.c32                         ├── ls.c32
    ├── lua.c32                        ├── lua.c32
    ├── mboot.c32                      ├── mboot.c32
    ├── meminfo.c32                    ├── meminfo.c32
    ├── menu.c32                       ├── menu.c32
    ├── pci.c32                        ├── pci.c32
    ├── pwd.c32                        ├── pwd.c32
    ├── reboot.c32                     ├── reboot.c32
    ├── rosh.c32                       ├── rosh.c32
    ├── sysdump.c32                    ├── sysdump.c32
    ├── syslinux.c32                   ├── syslinux.c32
    ├── syslinux.cfg *                 ├── syslinux.cfg *
    ├── vesa.c32                       ├── vesa.c32
    ├── vesamenu.c32                   ├── vesamenu.c32
    ├── vpdtest.c32                    ├── vpdtest.c32
    ├── whichsys.c32                   ├── whichsys.c32
    └── zzjson.c32                     └── zzjson.c32
```

* Duplicate files with equivalent contents. Although all files are duplicates, you should pay attention especially to these files. 

Basically, you need:

- A) existing `syslinux` installation

- B) Copy files from `/usr/lib/syslinux/efi64/` to the `EFI/BOOT/` and `EFI/syslinux/` subfolders presented above.

Be aware: `.c32` files presented here may vary depending on version of your `syslinux` package. It should be fine, though.

### Boot partition permissions

**NOTE:** You don't need to set these manually if you use proper `/etc/fstab` entry mask values as defined in [fstab configuration file](#fstab-configuration-file). These values are here so you can just check them.

|   Permissions    | Owner (user:group) |   Type    |             Item Name              |
|------------------|--------------------|-----------|------------------------------------|
| rwxr-xr-x (0755) | root:root          | Directory | ./EFI                              |
| rwxr-xr-x (0755) | root:root          | Directory | ./EFI/BOOT                         |
| rwxr-xr-x (0755) | root:root          | Directory | ./EFI/syslinux                     |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/initramfs-linux-fallback.img |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/initramfs-linux.img          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/vmlinuz-linux                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/intel-ucode.img              |
| ---------------- | ---------          | --------- | ---------------------------------- |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/bootx64.efi             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cat.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/chain.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cmd.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cmenu.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/config.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cptime.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cpu.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cpuid.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/cpuidtest.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/debug.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/dhcp.c32                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/dir.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/dmi.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/dmitest.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/gfxboot.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/hdt.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/hexdump.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/host.c32                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/ifcpu64.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/ifcpu.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/ldlinux.e64             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/lfs.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/libcom32.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/libgpl.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/liblua.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/libmenu.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/libutil.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/linux.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/ls.c32                  |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/lua.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/mboot.c32               |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/meminfo.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/menu.c32                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/pci.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/pwd.c32                 |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/reboot.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/rosh.c32                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/sysdump.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/syslinux.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/syslinux.cfg            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/vesa.c32                |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/vesamenu.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/vpdtest.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/whichsys.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/BOOT/zzjson.c32              |
| ---------------- | ---------          | --------- | ---------------------------------- |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cat.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/chain.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cmd.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cmenu.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/config.c32          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cptime.c32          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cpu.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cpuid.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/cpuidtest.c32       |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/debug.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/dhcp.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/dir.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/dmi.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/dmitest.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/gfxboot.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/hdt.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/hexdump.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/host.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/ifcpu64.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/ifcpu.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/ldlinux.e64         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/lfs.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/libcom32.c32        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/libgpl.c32          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/liblua.c32          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/libmenu.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/libutil.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/linux.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/ls.c32              |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/lua.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/mboot.c32           |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/meminfo.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/menu.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/pci.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/pwd.c32             |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/reboot.c32          |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/rosh.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/sysdump.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/syslinux.c32        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/syslinux.cfg        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/syslinux.efi        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/vesa.c32            |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/vesamenu.c32        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/vpdtest.c32         |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/whichsys.c32        |
| rw-r--r-- (0644) | root:root          | File      | ./EFI/syslinux/zzjson.c32          |

## Filesystem structure

### Mount points overview

|  Item Name  |                 Item Type                    |       Partition       | Partition format |
|-------------|----------------------------------------------|-----------------------|------------------|
| /bin        | Symbolic link (to usr/bin)                   | NVMe, 2 (OS)          | Ext4             |
| /boot ***** | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /boot_efi * | Directory                                    | NVMe, 1 (Boot)        | FAT32/EFI        |
| /dev        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /etc        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /home       | Directory (bind) **                          | HDD/SSD (User files)  | Ext4             |
| /init       | Symbolic link (to `usr/lib/systemd/systemd`) | NVMe, 2 (OS)          | Ext4             |
| /lib        | Symbolic link (to `usr/lib`)                 | NVMe, 2 (OS)          | Ext4             |
| /lib64      | Symbolic link (to `usr/lib`)                 | NVMe, 2 (OS)          | Ext4             |
| /lost+found | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /mnt ***    | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /opt        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /proc       | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /root       | Directory (bind) **                          | HDD/SSD (User files)  | Ext4             |
| /run        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /sbin       | Symbolic link (to `usr/bin`)                 | NVMe, 2 (OS)          | Ext4             |
| /swap ****  | Directory (bind) **                          | HDD/SSD (Cache files) | Ext4             |
| /srv        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /sys        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /tmp        | Directory                                    | RAM                   | tmpfs            |
| /usr        | Directory                                    | NVMe, 2 (OS)          | Ext4             |
| /var        | Directory (bind) **                          | HDD/SSD (Cache files) | Ext4             |

* Mount point `/boot_efi` for UEFI boot partition is defined in `/etc/fstab`. Please see [fstab configuration file](#fstab-configuration file) for additional information.

** `bind` option is defined in `/etc/fstab`.  Please see the following diagram.

*** Directory `/mnt` contains directories existing on HDD/SSD (User files). Please see the following diagram.

**** Optional: non-standard location for a SWAP file. Please see [Arch Wiki - Swap file](https://wiki.archlinux.org/index.php/Swap#Swap_file) for additional information.

***** `/boot` folder contains only `intel-ucode.img` file, provided by Arch Linux package `intel-ucode`. Otherwise, `/boot` folder is empty and all necessary boot files are in `/boot_efi`

### `/mnt` location overview

|      Item Name       |           Item Type           | Partition | Partition format |
|----------------------|-------------------------------|-----------|------------------|
| /mnt/hdd             | Directory                     | HDD/SSD   | Ext4             |
| /mnt/hdd/home        | Directory (binded to `/home`) | HDD/SSD   | Ext4             |
| /mnt/hdd/home/<user> | Directory                     | HDD/SSD   | Ext4             |
| /mnt/hdd/home/root   | Directory (binded to `/root`) | HDD/SSD   | Ext4             |
| /mnt/hdd/var         | Directory (binded to `/var`)  | HDD/SSD   | Ext4             |
| /mnt/hdd/lost+found  | Directory                     | HDD/SSD   | Ext4             |
| /mnt/hdd/swap        | Directory (binded to `/swap`) | HDD/SSD   | Ext4             |

`<user>` refers to your Linux file system user account(s).

## fstab configuration file

The following `/etc/fstab` configuration corresponds to the file system structure explained above.

Please configure `UUID` values for your setup. You must use proper `UUID` values instead of the ones presented below (see [Retrieving UUIDs](#retrieving-uuids)). Additionally, use proper username string for `<user>`.

```
> cat /etc/fstab

# 
# /etc/fstab: static file system information
#
# <file system>                                 <dir>       <type>      <options>                                   <dump>  <pass>
#
#########################################
# NVMe: M.2 - FILE SYSTEM PARTITION

# Root system
UUID=9148225b-d661-4e4a-801c-f5bdc48f509e       /           ext4        defaults,noatime,discard                    0       1

#########################################
# Boot files - FAT32 partition, needs mask values

UUID=927D-38E4                                  /boot_efi   vfat        defaults,dmask=0022,fmask=0133              0       0

#########################################
# RAM & SWAP

# temporary folders
tmpfs                                           /tmp        tmpfs       nodev,nosuid,size=20480M,nr_inodes=500k     0       0

# Swap memory file (optional)
/mnt/hdd/swap                                   /swap       none        bind                                        0       0
/swap/swapfile                                  none        swap        defaults                                    0       0

#########################################
# HDD: SATA - USER FILES PARTITION

UUID=fba31569-c25c-4e30-b117-9ea83c41a8bd       /mnt/hdd     ext4       defaults,discard                            0       2

#########################################
# Bind folders

# /var folder
/mnt/hdd/var                                    /var        none        bind                                        0       0

# Home folders

/mnt/hdd/home/root                              /root       none        bind                                        0       0
/root                                           /home/root  none        bind                                        0       0

#########################################

# Optional: mount your cache folders to RAM

#tmpfs                                           /home/<user>/.cache tmpfs size=15%,mode=0777,uid=1000,gid=1000     0       0
```

## syslinux configuration file

This configuration applies to the files `EFI/BOOT/syslinux.cfg` & `EFI/syslinux/syslinux.cfg` on NVMe partition 1 (Boot). Please configure `APPEND` kernel parameters properly for your setup. At least, you must use proper `UUID` values instead of the ones presented below (see [Retrieving UUIDs](#retrieving-uuids)).

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
TIMEOUT 5
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
#

LABEL arch
    MENU LABEL Arch Linux
    LINUX ../vmlinuz-linux
    INITRD ../initramfs-linux.img
    APPEND root=UUID=9148225b-d661-4e4a-801c-f5bdc48f509e rw vga=895 acpi_osi="!Windows 2015" acpi_osi=Linux nowatchdog nogpumanager rd.udev.log-priority=3 vm.swappiness=10 net.ifnames=0 biosdevname=0 cgroup_no_v1=all

LABEL archshell
    MENU LABEL Arch Linux (shell)
    LINUX ../vmlinuz-linux
    INITRD ../initramfs-linux.img
    APPEND root=UUID=9148225b-d661-4e4a-801c-f5bdc48f509e rw vga=895 systemd.unit=multi-user.target nowatchdog vm.swappiness=10 net.ifnames=0 biosdevname=0 cgroup_no_v1=all

LABEL archfallback
    MENU LABEL Arch Linux Fallback
    LINUX ../vmlinuz-linux
    INITRD ../initramfs-linux-fallback.img
    APPEND root=UUID=9148225b-d661-4e4a-801c-f5bdc48f509e rw vga=895 acpi_osi="!Windows 2015" acpi_osi=Linux nowatchdog vm.swappiness=10 net.ifnames=0 biosdevname=0

LABEL archfallback-rescue
    MENU LABEL Arch Linux Fallback (rescue mode)
    LINUX ../vmlinuz-linux
    INITRD ../initramfs-linux-fallback.img
    APPEND root=UUID=9148225b-d661-4e4a-801c-f5bdc48f509e vga=895 systemd.unit=multi-user.target nowatchdog vm.swappiness=10 net.ifnames=0 biosdevname=0

#LABEL windows
#        MENU LABEL Windows
#        COM32 chain.c32
#        APPEND hd0 1

## These do not work with UEFI
#LABEL hdt
#        MENU LABEL HDT (Hardware Detection Tool)
#        COM32 hdt.c32

#LABEL reboot
#        MENU LABEL Reboot
#        COM32 reboot.c32

#LABEL poweroff
#        MENU LABEL Poweroff
#        COM32 poweroff.c32
```

## mkinitcpio preset file

Custom `/etc/mkinitcpio.d/linux.preset` file is recommended for this setup. Each time you run `mkinitcpio -p linux` or update Linux kernel or systemd configuration, proper initrd and kernel files are automatically generated in proper location (`/boot_efi/` in this setup) without additional hassle.

```
> cat /etc/mkinitcpio.d/linux.preset

# mkinitcpio preset file for the 'linux' package

ESP_DIR="/boot_efi/EFI"

ALL_config="/etc/mkinitcpio.conf"
ALL_kver="${ESP_DIR}/vmlinuz-linux"

PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
default_image="${ESP_DIR}/initramfs-linux.img"
default_options="-A esp-update-linux"

#fallback_config="/etc/mkinitcpio.conf"
fallback_image="${ESP_DIR}/initramfs-linux-fallback.img"
fallback_options="-S autodetect"
```

## mkinitcpio configuration file

The following `/etc/mkinitcpio.conf` file works in this setup. The file is used to configure initial RAM disk files (`initramfs.img`, `initramfs-fallback.img`). Not all modules listed in `MODULES` array are required, but they have been left for [Optimus/Intel GPU passthrough setup](https://gist.github.com/agentsim/e89beb42ede85714a24529b2a6798bb8) I once had.

```
# vim:set ft=sh
# MODULES
# The following modules are loaded before any boot hooks are
# run.  Advanced users may wish to specify all system modules
# in this array.  For instance:
#     MODULES=(piix ide_disk reiserfs)
MODULES=(vfio vfio_iommu_type1 vfio_pci vfio_virqfd vhost-net pci-stub)

# BINARIES
# This setting includes any additional binaries a given user may
# wish into the CPIO image.  This is run last, so it may be used to
# override the actual binaries included by a given hook
# BINARIES are dependency parsed, so you may safely ignore libraries
BINARIES=()

# FILES
# This setting is similar to BINARIES above, however, files are added
# as-is and are not parsed in any way.  This is useful for config files.
FILES=()

# HOOKS
# This is the most important setting in this file.  The HOOKS control the
# modules and scripts added to the image, and what happens at boot time.
# Order is important, and it is recommended that you do not change the
# order in which HOOKS are added.  Run 'mkinitcpio -H <hook name>' for
# help on a given hook.
# 'base' is _required_ unless you know precisely what you are doing.
# 'udev' is _required_ in order to automatically load modules
# 'filesystems' is _required_ unless you specify your fs modules in MODULES
# Examples:
##   This setup specifies all modules in the MODULES setting above.
##   No raid, lvm2, or encrypted root is needed.
#    HOOKS=(base)
#
##   This setup will autodetect all modules for your system and should
##   work as a sane default
#    HOOKS=(base udev autodetect block filesystems)
#
##   This setup will generate a 'full' image which supports most systems.
##   No autodetection is done.
#    HOOKS=(base udev block filesystems)
#
##   This setup assembles a pata mdadm array with an encrypted root FS.
##   Note: See 'mkinitcpio -H mdadm' for more information on raid devices.
#    HOOKS=(base udev block mdadm encrypt filesystems)
#
##   This setup loads an lvm2 volume group on a usb device.
#    HOOKS=(base udev block lvm2 filesystems)
#
##   NOTE: If you have /usr on a separate partition, you MUST include the
#    usr, fsck and shutdown hooks.
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)

# COMPRESSION
# Use this to compress the initramfs image. By default, gzip compression
# is used. Use 'cat' to create an uncompressed image.
#COMPRESSION="gzip"
#COMPRESSION="bzip2"
#COMPRESSION="lzma"
#COMPRESSION="xz"
#COMPRESSION="lzop"
#COMPRESSION="lz4"

# COMPRESSION_OPTIONS
# Additional options for the compressor
#COMPRESSION_OPTIONS=()

```

--------------------------

## Additional hints

### Retrieving UUIDs

Run `blkid` command as root or with `sudo`.

--------------------------

## RewriteFS

RewriteFS is a solution to force writing of all hidden [dot files]() into `$HOME/.config` location. Of course, this is just a default setup and can be configured further.

RewriteFS is available in AUR as [rewritefs-git](https://aur.archlinux.org/packages/rewritefs-git/) package.

**WARNING:** Using `rewritefs` may increase your HDD usage in heavy file operations.

### RewriteFS - installation

**1)** Install RewriteFS (AUR: [rewritefs-git](https://aur.archlinux.org/packages/rewritefs-git/))

**2)** Move all your dotfiles and dotfolders from `$HOME` to `$HOME/.config/` folder (excluding `.config`, `.cache` & `.local` folders). Remove the dot prefix from the names of those moved dotfiles/folders. You should end up of having only three dotfolders in your `$HOME` directory: `.cache`, `.config` and `.local` (and no dotfiles at all). All your previous dotfiles/folders are now in `$HOME/.config`, without the dot prefix in their name. Right? Good. Go ahead.

**3)** Add file `/etc/rewritefs.conf` with the following contents:

```
m#^(?!\.)# .
m#^\.(cache|config|local)# .
m#^\.# .config/
```

**4)** Uncomment `user_allow_other` in `/etc/fuse.conf`. According to the rewritefs author, you must do it:

```
The files are owned by the user of the process that create them, which is rewritefs, not your applications. You have to run rewritefs as [user], not root.
```

**5)** Configure your fstab. Use `/etc/fstab` example below as a configuration reference.

**NOTE:** It is important to set `<user>` UID and `<user>` GID bits in `/etc/fstab`. Otherwise you may end up creating new files which belong to group root instead of your actual primary group in `$HOME`.

**6)** Log out, switch to another TTY and login as another user or alternatively unmount the partition where your `$HOME` locates at.

**7)** Reboot the system

### RewriteFS - fstab example

```
> cat /etc/fstab

# 
# /etc/fstab: static file system information
#
# <file system>                                 <dir>       <type>      <options>                                   <dump>  <pass>
#
#########################################
# NVMe: M.2 - FILE SYSTEM PARTITION

# Root system
UUID=9148225b-d661-4e4a-801c-f5bdc48f509e       /           ext4        defaults,noatime,discard                    0       1

#########################################
# Boot files - FAT32 partition, needs mask values

UUID=927D-38E4                                  /boot_efi   vfat        defaults,dmask=0022,fmask=0133              0       0

#########################################
# RAM & SWAP

# temporary folders
tmpfs                                           /tmp        tmpfs       nodev,nosuid,size=20480M,nr_inodes=500k     0       0

# Swap memory file (optional)
/mnt/hdd/swap                                   /swap       none        bind                                        0       0
/swap/swapfile                                  none        swap        defaults                                    0       0

#########################################
# HDD: SATA - USER FILES PARTITION

UUID=fba31569-c25c-4e30-b117-9ea83c41a8bd       /mnt/hdd     ext4       defaults,discard                            0       2

#########################################
# Bind folders

# /var folder
/mnt/hdd/var                                    /var        none        bind                                        0       0

# Home folders

/mnt/hdd/home/root                              /root       none        bind                                        0       0
/root                                           /home/root  none        bind                                        0       0

/mnt/hdd/home/<user>                            /home/<user>     rewritefs       config=/etc/rewritefs.conf,allow_other,uid=1000,gid=1000      0       0

#########################################

# Optional: mount your cache folders to RAM

NOTE: When rewritefs is being used, mount order must be /tmp and then /home folders. This is because contents of user's .cache folder is written to /tmp/user_cache folder

#tmpfs                                           /home/<user>/.cache tmpfs size=15%,mode=0777,uid=1000,gid=1000     0       0
```

```
> stat -c "%A %G:%U %n" /etc/rewritefs.conf

-rw-r--r-- root:root /etc/rewritefs.conf
```

Common syntax for RewriteFS fstab entry is as follows (not related in this setup):

```
/mnt/home/<user> /home/<user> rewritefs config=/etc/rewritefs.conf,allow_other,uid=<user UID>,gid=<user GID> 0 0
```
