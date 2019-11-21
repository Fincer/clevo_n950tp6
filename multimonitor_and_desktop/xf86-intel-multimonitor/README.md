## Required boot parameters

The following kernel parameters are for correct GPU handling (Clevo N950TP6) on older Linux kernels. Without this setting you _may_ be unable to boot on Linux desktop on Clevo N950TP6 laptop.

Add these to your syslinux or GRUB boot parameters, either `/boot/syslinux/syslinux.cfg` OR `/boot/grub/grub.cfg` (GRUB configuration file `/etc/default/grub` preferred):

```
acpi_osi="!Windows 2015" acpi_osi=Linux nogpumanager
```

-------------------------

## About

This `multimonitor` package creates two desktop entries to turn on/off `intel-virtual-output` executable (can be run as daemon, too). When enabled, the program creates new VirtualGL devices which is seen by issuing `xrandr` command. These devices are referring to your external monitors you use with Clevo N950TP6 laptop. Thus, you can set up all needed configuration for these external monitors in your desktop settings.

Only downside with setting up new VirtualGL devices is that discrete Nvidia card must be enabled as well. At least on Linux, all external graphics output goes through Nvidia chipset on Clevo N950TP6. However, if you have your laptop plugged in to AC power adapter, you don't need to worry about Nvidia/Intel dilemma. Temperature of Nvidia card stays at ~47-53 while idle. It is higher on load, of course.

Please read a personal usage story below for additional information.

## Overriding configuration files

This package overrides the following bumblebee files:

```
/etc/bumblebee/xorg.conf.nvidia
/etc/bumblebee/xorg.conf.nouveau
```

and adds the following X11 configuration files:

```
/etc/X11/xorg.conf.d/10-intel.conf
/etc/X11/xorg.conf.d/20-nvidia.conf
/etc/X11/nvidia-xorg.conf
```

Therefore, you MUST install this package AFTER you have installed bumblebee. And you need to force installation with pacman parameter `--force` (`sudo pacman -U --force package.tar.xz`). Proper way to install these overriding files would be patching the corresponding packages. 

## Extra

Subfolder extra contains the default `xorg.conf` (`/etc/X11/xorg.conf`) file I personally use on Clevo N950TP6. You can use it if you find it useful.

-------------------------

## Multimonitor - Personal Story

There are two ways I personally use Nvidia card on my Clevo laptop. They are as follows:

- **1)** Multimonitor mode: enable additional display/screen output(s). This is which `intel-virtual-output` executable is for. I use this mode to enable one of the miniDisplayPort output, and attaching one additional monitor to my laptop. Another miniDisplayPort is enabled by default.

- **2)** Nvidia "only" mode: Switch to another TTY session (without X server enabled, only CLI). Start a new X server instance with Openbox DE by issuing command: `nvidia-xrun openbox-session` ([AUR - nvidia-xrun](https://aur.archlinux.org/packages/?O=0&SeB=nd&K=nvidia-xrun&outdated=&SB=n&SO=a&PP=50&do_Search=Go))

Worth noting: These modes can not be run simultaneously. For instance, I need to shut down `intel-virtual-output` process (1) before attempting to launch another X server session (2). Vice versa, I can't execute `intel-virtual-output` if I have a X server session already opened for Nvidia card (2).

In this way, I can control when Nvidia card is turned ON/OFF.

I use mode 1) for basic desktop tasks, and mode 2) for gaming (Steam, GOG, etc.)

Sometimes `intel-virtual-output` can complain about not finding bumblebee. You can try checking your `/etc/X11` conf files.

About role of bumblebee: it is required by `intel-virtual-output` binary. Practically, I don't use bumblebee for *anything* else.
