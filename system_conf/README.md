## Ethernet

If your `eth0` interface is not listed in `ip addr` output, you may consider blacklisting default `r8169` kernel module and installing `r8168` as a replacement module. 

**1)** Add `blacklist r8169` into `/etc/modprobe.d/ethernet.conf`:

```
echo "blacklist r8169" | tee -a /etc/modprobe.d/ethernet.conf
 
```

**2)** Install [r8168-dkms](https://aur.archlinux.org/packages/r8168-dkms/) from AUR repository

**3)** If you have `r8169` module loaded (`lsmod`), remove it with `rmmod r8169` and load installed `r8168` module (`modprobe r8168`)

## Nvidia Optimus

- Follow [Arch Wiki - Bumblebee](https://wiki.archlinux.org/index.php/Bumblebee) instructions

- Use Xorg configuration files provided [here](../multimonitor_and_desktop/xf86-intel-multimonitor)

- Blacklist several kernel modules:

```
> cat /etc/modprobe.d/nvidia.conf 

blacklist nvidia
blacklist nouveau
options bbswitch load_state=0
```
