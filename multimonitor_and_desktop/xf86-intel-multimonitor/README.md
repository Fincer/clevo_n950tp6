### Add the following boot parameters for syslinux/grub for correct GPU handling (Clevo N950TP6)

Add these to your boot parameters, either `/boot/syslinux/syslinux.cfg` OR `/boot/grub/grub.cfg` (GRUB configuration file `/etc/default/grub` preferred!):

```
acpi_osi="!Windows 2015" acpi_osi=Linux nogpumanager
```

**NOTE:** Without this setting you may be unable to boot on Linux desktop on Clevo N950TP6 laptop.

-------------------------

This `multimonitor` package creates two desktop entries to turn on/off `intel-virtual-output` executable (can be run as daemon, too). When enabled, the program creates new VirtualGL devices for `xrandr`. These devices are referring to your external monitors you use with Clevo N950TP6 laptop. Thus, you can set up all needed configuration for these external monitors in your desktop settings.

Only downside with setting up new VirtualGL devices is that discrete Nvidia card must be enabled as well. At least on Linux, all external graphics output goes through Nvidia chipset on Clevo N950TP6. However, if you have your laptop plugged in to AC power adapter, you don't need to worry about Nvidia/Intel dilemma. Temperature of Nvidia card stays at ~47-53 while idle. It is higher on load, of course.

**NOTE:**

This package overrides the following bumblebee file:

```
/etc/bumblebee/xorg.conf.nvidia
```

and adds the following configuration file:

```
/etc/X11/xorg.conf.d/20-intel.conf
```

Therefore, you MUST install this package AFTER you have installed bumblebee. And you need to force installation with pacman parameter `--force` (`sudo pacman -U --force package.tar.xz`)
