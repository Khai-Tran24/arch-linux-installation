# arch-linux-installation
My arch linux install process

## Step 1: Install Arch Linux ISO and verify its.
Download link: [Arch Linux](https://archlinux.org/download/)

- Install .iso and .iso.sig file

- Verify with GnuPG (remember to install GnuPG) 

- The download the signature and verify it 

```bash
gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org

gpg --verify archlinux-2026.02.01-x86_64.iso.sig archlinux-2026.02.01-x86_64.iso
```
  
## Step 2: Install Ventoy.
Download Link: [Ventoy](https://www.ventoy.net/en/download.html)

- After download ventoy, run it download file and install ventoy to USB.

- Then, move .iso file to the USB and Reboot (Keep press F12 to get into boot mode)

## Step 3: Download Arch Linux

- After reboot, select the arch linux boot and start install arch linux

### Step 3.1: Connect to wifi
- Check if the wifi is block by rfkill (if it is blocked, then unblock it)

```bash
rfkill

rfkill unblock wlan
```

- After unblock, start connecting wifi

```bash
iwctl
```

```bash
device list

station <device> scan

station <device> get-networks

station <device> connect <SSID> --ask
```
- Check the connection
```bash
ip link
```

### Step 3.2: Disk parition
- Use **gdisk** to partition the disk

```bash
gdisk /dev/nvme1n1
```

- n to create new partition, p to display partition list

| partition | first sector | last sector | code | name | required |
|-----------|--------------|-------------|------|------|----------|
| 1 | default | 512M | ef00 | EFI | yes |
| 2 | default | 1G | ef01 | BIOS | no |
| 3 | default | default | 8309 | LUKS | yes |

- You can create swap file during the partition step
- If you don't want to encrypt your disk, then you can use linux file instead of luks

### Step 3.3: Disk encryption
- Load encryption module 

```bash
modprobe dm-crypt

modprobe dm-mod
```

- Setting up encryption for disk

```bash
cryptsetup luksFormat -v -s 512 -h sha512 /dev/nvme1n1p3

cryptsetup open /dev/nvme1n1p3 luks_lvm
```

- Create volume for encryption disk

```bash
pvcreate /dev/mapper/luks_lvm

vgcreate arch /dev/mapper/luks_lvm
```

- Create volumes for swap and root

```bash
lvcreate -n swap -L 32G arch

lvcreate -n root -l +100FREE arch
```

### Step 3.4: Disk formation

```bash
mkfs.fat -F 32 /dev/nvme1n1p1

mkfs.ext4 /dev/nvme1n1p2

mkfs.btrfs -L root /dev/mapper/arch-root

mkswap /dev/mapper/arch-swap
```

### Step 3.5: Mounting 
- Mounting swap
```bash
swapon /dev/mapper/arch-swap

swapon -a
```

- Mounting other partitions
```bash
mount /dev/mapper/arch-root /mnt

mount /dev/nvme1n1p2 /mnt/boot

mkdir /mnt/boot/efi

mount /dev/nvme1n1p1 /mnt/boot/efi
```

### Step 3.6: Install Package
- Install arch linux package
```bash
pacstrap -K linux base-devel base linux-firmware

genfstab -U -p /mnt > /mnt/etc/fstab

arch-chroot /mnt /bin/bash
```

- Install necessary packages
- Instead of use grub for bootloader, you can use systemd bootloader
```bash
pacman -S neovim lvm2 grub efibootmgr networkmanager intel-ucode

systemctl enable NetworkManager
```

- Decrypting volumes when booting
```bash
nvim /etc/mkinitcpio.conf
```

- Check **HOOKS**,
```
HOOKS = (... systemd ... block sd-encrypt lvm2 filesystems ...)
```

- Setup bootloader
```bash 
grub-install --efi-directory=/boot/efi

nvim /etc/default/grub
```

- Check the encrypt partition id
```bash
blkid /dev/nvme1n1p3
```

- copy and paste the line below 
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet root=/dev/mapper/arch-root rd.luks.name=<uuid>=luks_lvm"
```

- Keyfile
```bask 
mkdir /secure

dd if=/dev/random of=/secure/root_keyfile.bin bs=512 count=8

chmod 000 /secure/*

cryptsetup luksAddKey /dev/nvme1n1p3 /secure/root_keyfile.bin

nvim /etc/mkinitcpio.conf

-- add keyfile to FILES --
FILES=(/secure/root_keyfile.bin)
```

- Create Grub config
```bash
grub-mkconfig -o /boot/grub/grub.cfg

grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
```

- Configure system
    - Time
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime

    nvim /etc/systemd/timesyncd.conf

    -- Add this in NTP servers --
    [Time]
    NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org 
    FallbackNTP=0.pool.ntp.org 1.pool.ntp.org

    systemctl enable systemd-timesyncd.service
    ```

    - Locale
    ```bash
    nvim /etc/locale.gen

    -- Uncomment --
    en_US.UTF-8 UTF-8

    locale-gen

    nvim /etc/locale.conf

    -- Input --
    LANG=en_US.UTF-8
    ```

    - Hostname
    ```bash
    nvim /etc/hostname
    -- Enter hostname you want --
    ```

    - Users
    ```bash
    passwd

    pacman -S zsh

    useradd -m -G wheel -s /bin/zsh <user>

    passwd <user>

    EDITOR=nvim

    visudo

    -- Uncomment --
    %wheel ALL=(ALL:ALL) ALL

    ```

```bash
grub-mkconfig -o /boot/grub/grub.cfg

grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg

mkinitcpio -p linux
```

- Reboot

```bash
exit

umount -R /mnt

reboot now
```

- After reboot, start install [Hyprland](https://wiki.hypr.land/Getting-Started/Installation/) and packages

## References
- [Arch Install](https://wiki.archlinux.org/title/Installation_guide)
- [mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio#Hook_list)
- [Dreams of Autonomy](https://www.youtube.com/watch?v=YC7NMbl4goo&t=673s)
