# clevo_n950tp6
Various instructions for setting up Linux OS on Clevo N950TP6

![img_1](https://github.com/Fincer/clevo_n950tp6/blob/master/images/linux-run/linux-run_3.jpg)
![img_2](https://github.com/Fincer/clevo_n950tp6/blob/master/images/general_5.jpg)

## Table of contents

----------------

**Good to know**

- Laptop bought from [Clevo Systems](http://clevo-systems.com/). Fast shipping and candies included. Recommended!

- Came with Windows 10 preinstalled on NVMe SSD (Samsung EVO 960) although I set "No OS" on their website.

- If you had Win10 preinstalled, back it up - create a complete backup of your new NVMe SSD. Just in case. I used `dd`

- You need Grub/Syslinux boot parameters `acpi_osi="!Windows 2015" acpi_osi=Linux` in order to access Linux desktop environment as recent Linux OSes tend to freeze on this laptop while loading DE. This rule applies to Live USB/CDs as well.

- You need a recent Linux kernel (4.14 >) for this laptop

- After proper configuration, everything works as expected. Recommended for Linux usage if you can deal with Nvidia Optimus

- If you don't use UEFI (read: you use traditional HDD instead of fast NVMe SSD), you need to disable `UEFI boot` option in UEFI/BIOS menu. You can re-enable it if you [decided to use UEFI boot](https://github.com/Fincer/clevo_n950tp6/blob/master/ssd_hdd_uefi/notes.md)

----------------

- **[CPU Thermal Control Configuration](https://github.com/Fincer/clevo_n950tp6/blob/master/cpu_thermal_control/notes.md)**

    - **What? :** Set a maximum temperature limit for your CPU (intel i7-8700)

    - [Files](https://github.com/Fincer/clevo_n950tp6/blob/master/cpu_thermal_control)

    - [PKGBUILD for Arch Linux](https://github.com/Fincer/clevo_n950tp6/blob/master/cpu_thermal_control/PKGBUILD)

----------------

- **[Laptop Images](https://github.com/Fincer/clevo_n950tp6/blob/master/images)**

  - **What? :** Sample images of Clevo N950TP6 laptop
  
  - [BIOS/UEFI screens](https://github.com/Fincer/clevo_n950tp6/tree/master/images/bios)
  
  - [Linux images](https://github.com/Fincer/clevo_n950tp6/tree/master/images/linux-run)
  
  - [General laptop images (enclosure etc.)](https://github.com/Fincer/clevo_n950tp6/tree/master/images)
  
  - [Physical interfaces/connections](https://github.com/Fincer/clevo_n950tp6/tree/master/images/inputs)

----------------

- **[Keyboard Backlight & Color Support](https://github.com/Fincer/clevo_n950tp6/blob/master/keyboard_color_support/notes.md)**

    - **What? :** Control laptop keyboard backlight & colors, patched for Clevo N950 TP6 (tested and works!)

    - [Files](https://github.com/Fincer/clevo_n950tp6/blob/master/cpu_thermal_control)

        - [clevo-xsm-wmi-dkms - PKGBUILD for Arch Linux](https://github.com/Fincer/clevo_n950tp6/blob/master/keyboard_color_support/clevo-xsm-wmi-dkms/PKGBUILD)

        - [clevo-xsm-wmi-util - PKGBUILD for Arch Linux](https://github.com/Fincer/clevo_n950tp6/blob/master/keyboard_color_support/clevo-xsm-wmi-util/PKGBUILD)

----------------

- **[Multi-monitor support](https://github.com/Fincer/clevo_n950tp6/blob/master/multimonitor_and_desktop/xf86-intel-multimonitor/notes.md)**

    - **What? :** Enable multi-monitor support for your Clevo N950TP6 laptop (Linux)

    - [Files](https://github.com/Fincer/clevo_n950tp6/tree/master/multimonitor_and_desktop/xf86-intel-multimonitor)

        - [xf86-intel-multimonitor - PKGBUILD for Arch Linux](https://github.com/Fincer/clevo_n950tp6/blob/master/multimonitor_and_desktop/xf86-intel-multimonitor/PKGBUILD)

----------------

- **[UEFI // SSD + HDD configuration](https://github.com/Fincer/clevo_n950tp6/blob/master/ssd_hdd_uefi/notes.md)**

    - **What? :** A sample configuration instructions for setting up UEFI and NVMe SSD + HDD on Clevo N950TP6 (Linux)
