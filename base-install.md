# Arch Linux Installation Guide
Windows 10/Arch dual boot on separate SSDs - UEFI - rEFInd boot manager


---
# USB
Grab .iso from https://www.archlinux.org/download/
install via Etcher

Boot into bios
disable secure boot
disable fast startup mode

restart spam f12 select usb

# Pre install

###### 1. Check for internet
```sh
$ ping google.com
or
$ ip link
```
###### 2. Check boot mode for UEFI - if directory does not exist boot mode is BIOS
```sh
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

Should look like this after creating partitions - check with `lsblk`
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
| sdc1 | `/root`          | 40G             | ext4 |
| sdc2 | `/home`          | 183G            | ext4 |

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
    # sdc1 is the root partition
    $ mount /dev/sdc1 /mnt
    ```
2. Mount /boot partition:
    ```sh
    # sdb2 is the windows EFI partition 
    $ mkdir /mnt/boot
    $ mount /dev/sdb2 /mnt/boot
    ```
3. Mount /home partition:
    ```sh
    # sdb2 is the home partition
    $ mkdir  /mnt/home
    $ mount /dev/sdc2 /mnt/home
    ```
Should look like this after mounting - check with `lsblk`
| Name | Partition        |  Size           | Mount Point |
| :--: | :-------:        | :-------------: | :--: 	  |
| `sda`| `hdd storage`    | 931G            | 		  |
| sda1 |                  | 16M             |      	  |
| sda2 |                  | 927.9G          |      	  |
| `sdb`| `Windows Drive`  | 232G            | 		  |
| sdb1 |                  | 450M            |      	  |
| sdb2 | `EFI`            | 100M            | /mnt/boot   |
| sdb3 |                  | 231.5G          |      	  |
| sdb4 |                  | 86M             |      	  |
|`sdc` | `arch disk`      | 223.6G          | 		  |
| sdc1 | `/root`          | 40G             | /mnt	  |
| sdc2 | `/home`          | 183G            | /mnt/home   |

---

### Installation 
###### Install base
```sh
$ pacstrap /mnt base linux linux-firmware nano
```
###### fstab
```sh
$ genfstab -U /mnt >> /mnt/etc/fstab
```
```sh
# check fstab
$ cat /mnt/etc/fstab
```
###### Chroot into system
```sh
$ arch-chroot /mnt
```
###### Create swap
```sh
$ fallocate -l #GB /swapfile
```
```sh
$ chmod 600 /swapfile
```
```sh
$ mkswap /swapfile
```
```sh
$ swapon /swapfile
```
```sh
$ nano /etc/fstab
```
```sh
# Add to bottom of fstab file 
$ /swapfile none swap defaults 00
```

### Configure System
###### Timezone
```sh
$ ln -sf /usr/share/zoneinfo/America/Los_Angeles  /etc/localtime
```
```sh
$ hwclock --systohc
```
###### Locales
```sh
$ nano /etc/locale.gen
```
```sh
# uncomment un_US.UTF-8 UTF8
```
```sh
$ locale-gen
```
```sh
$ nano /etc/locale/conf
```
```sh
# add 
$ LANG=en_US.UTF-8
```
###### Host Name
```sh
$ hostnamectl set-hostmane #name
```
```sh
$ nano /etc/hosts
```
```
#Add to hosts
# IPv4 Hosts
127.0.0.1	localhost
::1 		localhost
127.0.1.1 	#name.localdomain	#name

# Machine FQDN
127.0.1.1	myhostname.localdomain	myhostname

# IPv6 Hosts
::1		localhost	ip6-localhost	ip6-loopback
ff02::1 	ip6-allnodes
ff02::2 	ip6-allrouters
```
###### Set root password
```sh
$ passwd
```
---
### Boot Manager
###### rEFInd
```sh
$ pacman -S refind-efi efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog os-prober base-devel mtools dosfstools linux-headers
g
g
g
g
g
g
g
```
### Network Manager
###### Enable
```sh
$ systemctl enable NetworkManager
```
### Add user
###### 
```sh
$ useradd -mG wheel #user
```
```sh
$ EDITOR=nano visudo
```
```sh
$ #uncomment %wheel ALL=(ALL) ALL
```



## References
- https://wiki.archlinux.org/

