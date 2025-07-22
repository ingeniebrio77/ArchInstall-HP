# Guía de Instalación: Arch Linux Seguro y Moderno
**Versión: 1.0**
... (todo el resto del contenido) ...

## tutorial de Walian
[Walian Tech/Linux Blog](https://www.walian.co.uk/arch-install-with-secure-boot-btrfs-tpm2-luks-encryption-unified-kernel-images.html)

### Particionado de disco

`sgdisk -Z /dev/nvme0n1`
`sgdisk -n1:0:+1024M -t1:ef00 -c1:EFI -N2 -t2:8304 -c2:LINUXROOT /dev/nvme0n1`
`partprobe -s /dev/nvme0n1`
`lsblk /dev/nvme0n1`

### Root Partition + LUKS

`cryptsetup luksFormat --type luks2 /dev/nvme0n1p2`

`cryptsetup luksOpen /dev/nvm0n1p2 linuxroot

### Create Filesystems

`mkfs.vfat -F32 -n EFI /dev/nvme0n1p1`
`mkfs.btrfs -f -L linuxroot /dev/mapper/linuxroot`

### Encrypt Root Partition with LUKS

`cryptsetup luksFormat --type luks2 /dev/nvme0n1p2`

`cryptsetup luksOpen /dev/nvme0n1p2 linuxroot`

### Let's create the filesystems:

`mkfs.vfat -F32 -n EFI /dev/nvme0n1p1`
`mkfs.btrfs -f -L linuxroot /dev/mapper/linuxroot`

### Let's mount our partitions, and create our btrfs subvolumes

`mount /dev/mapper/linuxroot /mnt`
`mkdir /mnt/efi`
`mount /dev/nvme0n1p1 /mnt/efi`
`btrfs subvolume create /mnt/home`
`btrfs subvolume create /mnt/srv`
`btrfs subvolume create /mnt/var`
`btrfs subvolume create /mnt/var/log`
`btrfs subvolume create /mnt/var/cache`
`btrfs subvolume create /mnt/var/tmp`

### Base install

`reflector --country DE --age 24 --protocol http,https --sort rate --save /etc/pacman.d/mirrorlist`
`pacstrap -K /mnt base base-devel linux linux-firmware intel-ucode vim nano cryptsetup btrfs-progs dosfstools util-linux git unzip sbctl kitty networkmanager sudo openssh`

### Update our locale settings:

`sed -i -e "/^#"en_GB.UTF-8"/s/^#//" /mnt/etc/locale.gen`
`sed -i -e "/^#"de_DE.UTF-8"/s/^#//" /mnt/etc/locale.gen`
`sed -i -e "/^#"es_ES.UTF-8"/s/^#//" /mnt/etc/locale.gen`
`systemd-firstboot --root /mnt --prompt`
`arch-chroot /mnt locale-gen`

### User creation

`arch-chroot /mnt useradd -G wheel -m renato
`arch-chroot /mnt passwd renato`
`arch-chroot /mnt passwd`
`sed -i -e '/^# %wheel ALL=(ALL:ALL) NOPASSWD: ALL/s/^# //' /mnt/etc/sudoers`

### Unified Kernel fun

`echo "quiet rw nvidia_drm.modeset=1" > /mnt/etc/kernel/cmdline`
`mkdir -p /mnt/efi/EFI/Linux`

### Change HOOKS /mnt/etc/mkinitcpio.conf

`HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt resume filesystems fsck)`
`MODULES=(btrfs nvidia nvidia_modeset nvidia_uvm nvidia_drm)`
`arch-chroot /mnt pacman -S nvidia-dkms linux-headers`
`arch-chroot /mnt mkinitcpio -P`
`ls -lR /mnt/efi`

### Services and Boot Loader
`systemctl --root /mnt enable systemd-resolved systemd-timesyncd NetworkManager sshd`
`systemctl --root /mnt mask systemd-networkd`
`arch-chroot /mnt bootctl install --esp-path=/efi`

#### Reboot
`sync`
`systemctl reboot --firmware-setup`

### Secure Boot with TPM2 Unlocking

`sbctl status`
`sudo sbctl create-keys`
`sudo sbctl enroll-keys -m`
`sudo sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi`
`sudo sbctl sign -s /efi/EFI/BOOT/BOOTX64.EFI`
`sudo sbctl sign -s /efi/EFI/Linux/arch-linux.efi`
`sudo sbctl sign -s /efi/EFI/Linux/arch-linux-fallback.efi`
`sudo pacman -S linux`
`sudo systemd-cryptenroll /dev/gpt-auto-root-luks --recovery-key`
`sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7  /dev/gpt-auto-root-luks`
`reboot`

#### Al reiniciar aun pedia la contrasena
`sudo systemd-cryptenroll --wipe-slot=tpm2 --tpm2-device=auto --tpm2-pcrs=7 /dev/nvme0n1p2`
