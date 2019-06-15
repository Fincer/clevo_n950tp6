At first, install `clevo-xsm-wmi-dkms`, then `clevo-xsm-wmi-util`

Patches for Clevo N950TP6 laptop and orange color support have been added.

----------------------

After you have installed `clevo-xsm-wmi-util`, do the following in your terminal:

Enable Clevo XSM WMI service:

```
sudo systemctl enable clevo-xsm-wmi.service
sudo systemctl start clevo-xsm-wmi.service
```

----------------------

If you don't intend to reboot your computer, do the following in your current session:

```
sudo modprobe clevo-xsm-wmi
```

----------------------

Launch the Clevo keyboard configuration utility with sudo or graphical sudo (kdesu, gksu etc.):

```
sudo /usr/bin/clevo-xsm-wmi
```

You need root rights to alter keyboard lightning!

----------------------

**NOTE:** At the moment of writing this, only changing keyboard colors & turning on/off the lightning works. I could not set any modes or alter keyboard brightness via the utility.

**NOTE:**

I am using KDE desktop environment. Therfore, I use kdesu command in my `clevo-xsm-wmi-util.desktop` file. If you use GTK or other desktop environment, change the sudo part of the Exec part to suit your needs. For example, I have

```
Exec=kdesu /usr/bin/clevo-xsm-wmi
```
