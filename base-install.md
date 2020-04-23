# Arch Linux Installation Guide
This guide will show step-by-step how to Install Arch Linux on UEFI mode.

## Table of Contents
- Bootable Flash Drive
- BIOS
- Pre installation
  - Set Keyboard Layout
  - Check boot mode
  - Update System Clock
  - Internet Connection
    - DHCP
    - Wi-Fi
    - Wired Connection
  - Partitioning
    - Create Partitions
    - Format Partitions
    - Mount the file system
- Installation
  - Select Mirror
  - Install Base Packages
  - Generate fstab
  - Chroot
  - Check pacman keys
- Configure System
  - Locale and Language
    - Keymap
    - Timezone
    - Hardware Clock
  - Network
    - Hostname
    - Nameservers
    - Firewall
  - Blacklists
    - No Beep
    - No Watchdog
  - Initramfs
  - Set-up Wi-Fi
  - Bootloader
  - Root password
  - Xorg
  - Video
  - Audio
  - Users
  - Reboot
- Post installation
  - Window Manager
  - Network Manager and services
- Extras
  - Set-up TTF Fonts
  - Bluetooth Headphone

---

# Pre install

###### 1. Check for internet
```sh
$ ping google.com
or
$ ip link
```
###### 2. Check boot mode
```sh
# Check boot mode for UEFI - If directory does not exist boot mode is BIOS
$ ls /sys/firmware/efi/efivars
```
###### 3. Update system clock
```sh
$ timedatectl set-ntp true
```
---
# Start Main Install
## Partition Setup
Example before creating partitions:

| Name | Partition        |  Size           | Type |
| :--: | :-------:        | :-------------: | :--: |
| `sda`| `hdd storage`    | 931G            | Disk |
| sda1 |                  | 16M             |      |
| sda2 |                  | 927.9G          |      |
| `sdb`| `Windows Drive`  | 232G            | Disk |
| sdb1 |                  | 450M            |      |
| sdb2 | `EFI`            | 100M            | EFI  |
| sdb3 |                  | 231.5G          |      |
| sdb4 |                  | 86M             |      |
| sdc  | `arch disk`      | 223.6G          | Disk |

1. Select Arch drive and format 
    - `lsblk` to list all partitions
    - Select Arch drive `fdisk /dev/sdc`
    - Create GPT disk `g`
### Partitioning
1. Create /root partition
    - `n` to create a new partition
    - Partition Number: default
    - First Sector: default
    - Root size eg. `+40G`
    - Enter

2. Create /home partition
    - `n` to create a new partition
    - Partition Number: default
    - First Sector: default
    - Home size remainder of disk: Enter
    - Enter

3. Save partition setup`w`

Should look like this after creating partitions 
`lsblk`
| Name | Partition        |  Size           | Type |
| :--: | :-------:        | :-------------: | :--: |
| `sda`| `hdd storage`    | 931G            | Disk |
| sda1 |                  | 16M             |      |
| sda2 |                  | 927.9G          |      |
| `sdb`| `Windows Drive`  | 232G            | Disk |
| sdb1 |                  | 450M            |      |
| sdb2 | `EFI`            | 100M            | EFI  |
| sdb3 |                  | 231.5G          |      |
| sdb4 |                  | 86M             |      |
|`sdc` | `arch disk`      | 223.6G          | Disk |
| sdc1  | `/root`         | 223.6G          | Disk |
| sdc2  | `/home`         | 223.6G          | Disk |

---

### Format Partitions
###### 1. Format /root
```sh
$ mkfs.ext4 /dev/sdc1
```
###### 1. Format /home
```sh
$ mkfs.ext4 /dev/sdc2
```

### Mount file system
1. Mount /root partition:
    ```sh
    # sdc1 = /root partition
    $ mount /dev/sdc1 /mnt
    ```
2. Mount /boot partition:
    ```sh
    # sdb2 = windows EFI partition
    $ mkdir /mnt/boot
    $ mount /dev/sdb2 /mnt/boot
    ```
3. Mount /home partition:
    ```sh
    # sdb2 = /home partition
    $ mkdir  /mnt/home
    $ mount /dev/sdc2 /mnt/home
    ```

---

## Installation
Now we'll install arch on disk

### Select Mirror
Before installation, is recommended to select the best mirror servers.
So open the file `/etc/pacman.d/mirrorlist` (again, you can use `nano` or `vi` to do that) and move the best mirror to the top of the file.

> **Tip**: That [link](https://www.archlinux.org/mirrorlist/) generates a mirror list based on your location, you can use them as reference.

### Install Base Packages
Now that the mirrors are already set, use `pacstrap` to install the base package group:
```sh
# pacstrap /mnt base base-devel
```

### Generate fstab
Now you should generate the fstab with the `genfstab` script:
```sh
# genfstab -p /mnt >> /mnt/etc/fstab
```

> Optional: you can add `noatime` to the generated `fstab` file (on root and home partitions) to increase IO performance.

### Chroot
Now, we'll change root into the new system
```sh
# arch-chroot /mnt
```

> Now, if you want to install some package, do it with `pacman -S <package_name>`

### Check pacman keys
```sh
# pacman-key --init
# pacman-key --populate archlinux
```

---

## Configure System
### Locale and Language
Open the file `/etc/locale.gen` and uncomment your locale settings

After that, write your locale string to file `/etc/locale.conf`.
For example, if you've uncomment the line `en_GK.UTF-8 UTF-8`, now you will write `en_GK.UTF-8`
```sh
echo en_GK.UTF-8 > /etc/locale.conf
```

Then, generate locale settings by running:
```sh
# locale-gen
```

And export your locale string with:
```sh
# export LANG=en_GK.UTF-8  #-- as example
```

#### Keymap
Create the file `/etc/vconsole.conf` and write your console settings. For example:
```
KEYMAP=br-abnt2
FONT=lat0-16
FONT_MAP=
```

#### Timezone
Create a symbolic link with your timezone (to check available timezones, see the files/folders in `/usr/share/zoneinfo/`)
```sh
# ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```

#### Hardware Clock
```sh
# hwclock --systohc --utc
```

### Network
#### Hostname
```sh
# echo myhostname > /etc/hostname
```

> Change `myhostname` to your hostname (Computer Name)

After that, open the file `/etc/hosts` and write (remember to change the `myhostname` to your own)

```
# IPv4 Hosts
127.0.0.1	localhost myhostname

# Machine FQDN
127.0.1.1	myhostname.localdomain	myhostname

# IPv6 Hosts
::1		localhost	ip6-localhost	ip6-loopback
ff02::1 	ip6-allnodes
ff02::2 	ip6-allrouters
```

#### Nameservers
Check the DNS again (using Google DNS). Open `/etc/resolv.conf` and write:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
```

#### Firewall
Write to file `/etc/modules-load.d/firewall.conf`:
```
# iptables modules to run on boot
ip_tables
nf_conntrack_netbios_ns
nf_conntrack
```

### Blacklists
**Warning**: this part is optional.

#### No Beep
To avoid the beep on boot, Write to file `/etc/modprobe.d/nobeep.conf`:
```
# Dont run pcpkr module on boot
blacklist pcspkr
```

#### No watchdog
If you don't want a watchdog service running, write to file `/etc/modprobe.d/nowatchdog.conf`
```
blacklist iTCO_wdt
```

### Initramfs
```sh
# mkinitcpio -p linux
```

### Set-up Wi-Fi
Install required packages with `pacman`
```sh
# pacman -S wireless_tools wpa_supplicant dialog
```

Now enable wireless connection automatically on system boot (it will be disabled later)

1. Go to `/etc/netctl` (with `cd` command)
1. List profiles with `netctl list`
1. Enable wifi-menu to automatically connect on boot:
    ```sh
    # netctl enable wlp1s0-MyWiFi
    ```

### Bootloader
Install Grub and efibootmgr:
```sh
# pacman -S grub efibootmgr
```

Run grub automatic installation on disk:
```sh
# grub-install /dev/sda
```

Create grub.cfg file:
```sh
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Root password
```sh
# passwd
```

### Xorg
Install Xorg Server: (use default options)
```sh
# pacman -S xorg-server
```

Define your keyboard layout on `/etc/X11/xorg.conf.d/10-keyboard.conf` file:
```
Section "InputClass"
	Identifier "keyboard default"
	MatchIsKeyboard "yes"
	Option  "XkbLayout" "br"
	Option  "XkbVariant" "abnt2"
EndSection
```

### Video
Install your GPU driver
```sh
# pacman -S xf86-video-vesa
```

### Audio
Install audio driver
```sh
# pacman -S alsa-utils
```

Configure and save:
```sh
# alsamixer
# alsactl store
```

### Users
Install sudo package
```sh
# pacman -S sudo
```

Configure sudo (uses `vim` as default editor) by running `visudo` and uncommenting the line:
```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

Now we're going to add a new user by running: (change `myuser` to your username)
```sh
# useradd -m -g users -G wheel myuser
```

Change the new user passord:
```sh
# passwd myuser
```

### Reboot
Exit chroot environment by pressing Ctrl + D or typing `exit`

Unmount system mount points:
```sh
# umount -R /mnt
```

Reboot system:
```sh
# reboot
```

> Remember to remove USB stick on reboot

---

## Post Installation
Now you're on your successfull Arch Linux installation.

Login with your user and follow the next steps.

### Window Manager
Now We're gonna install the Window Manager.

I'll show the steps to install [Gnome](https://www.gnome.org/).

First of all, run the installation command with `pacman`:
```sh
$ sudo pacman -S gnome gnome-extra
```

When the installation finishes, enable `gdm` to be started with system on boot:
```sh
$ sudo systemctl enable gdm.service
```

### Network Manager and services
Now we'll remove the previously enabled service from `netctl` and the `wifi-menu` settings.

First ensures that the NetworkManager package is installed:
```sh
$ sudo pacman -S networkmanager
```

Enable and start NetworkManager service:
```sh
$ sudo systemctl enable NetworkManager.service
$ sudo systemctl start NetworkManager.service
```

Go to `/etc/netctl` folder and see the connection files (the ones that starts with something like `wlp1s0...`)

Disable the netctl service that you've been enable previously:
```sh
$ sudo netctl diable wlp1s0-MyWiFi
```

Then, remove all `/etc/netctl` folder and remove your connection file (the one that starts with something like `wlp1s0...`)
```sh
$ sudo rm wlp1s0...  #-- replace with you wifi connection file
```

Now you can reboot the system (by running `reboot`) and everyting should be working fine.

---

## Extras
### Set-up TTF Fonts
Follow [this tutorial](https://gist.github.com/cryzed/e002e7057435f02cc7894b9e748c5671)

### Bluetooth Headphone
To connect the headphone:

1. Install required packages:
    ```sh
    $ sudo pacman -S pulseaudio pulseaudio-bluetooth pavucontrol bluez-utils
    ```
1. Edit `/etc/pulse/system.pa` and add:
    ```sh
    load-module module-bluez5-device
    load-module module-bluez5-discover
    ```
1. For GNOME users:
    ```sh
    $ sudo mkdir -p ~gdm/.config/systemd/user
    $ ln -s /dev/null ~gdm/.config/systemd/user/pulseaudio.socket
    ```
1. Connect to bluetooth device
    ```sh
    $ bluetoothctl
    # power on
    # agent on
    # default-agent
    # scan on
    # pair HEADPHONE_MAC
    # trust HEADPHONE_MAC
    # connect HEADPHONE_MAC
    # quit
    ```
To auto switch to A2DP mode:

1. Edit `/etc/pulse/default.pa` and add:
    ```
    .ifexists module-bluetooth-discover.so
    load-module module-bluetooth-discover
    load-module module-switch-on-connect  # Add this line
    .endif
    ```
1. Modify (or create) `/etc/bluetooth/audio.conf` to auto select AD2P profile:
    ```
    [General]
    Disable=Headset
    ```
1. Reboot PC to apply changes


## References
- https://wiki.archlinux.org/index.php/installation_guide
- https://forum.archlinux-br.org/viewtopic.php?id=4453
- https://itsfoss.com/install-arch-linux/
- https://www.ostechnix.com/install-arch-linux-latest-version/
- https://github.com/jieverson/dotfiles/wiki/arch-linux-for-dummies
