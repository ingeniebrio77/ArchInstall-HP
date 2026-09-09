# Guía de Instalación: Arch Linux Seguro y Moderno (HP Desktop M01-F1xxx)
**Versión: 2.1 (reajuste DE/trilingüe 2026-09-09: reflector timer con `--country DE`,
timezone `Europe/Berlin`, locales de/en/es, `KEYMAP=us`, `LANG=de_DE.UTF-8`)**
**Autor: renato — primera instalación Arch, documentada sobre la marcha**

> Esta guía fusiona las notas originales de instalación (`notas-originales-part1.md`)
> con la Parte 2 redactada (`Readmi.md`), corrigiendo typos, comandos rotos y
> contradicciones, y añade la Parte 3 (UKI + Secure Boot + TPM2) reconstruida a
> partir del estado real del equipo.
>
> Tutorial base: [Walian — Arch install with Secure Boot, Btrfs, TPM2, LUKS, UKI](https://www.walian.co.uk/arch-install-with-secure-boot-btrfs-tpm2-luks-encryption-unified-kernel-images.html)
> Solo ese tutorial + esta guía. Nada más.

**Resultado verificado:** systemd-boot 257.7 · Secure Boot `enabled (user)` con claves propias (sbctl) ·
desbloqueo LUKS por TPM2 · UKIs firmadas (`arch-linux.efi`, `arch-linux-fallback.efi`) ·
btrfs + LUKS2 · NVMe Samsung 990 PRO 1TB.

---

## Parte 0 — Preparación y BIOS

1. Arrancar la ISO de Arch (`archlinux-2026.01.01` o posterior) en modo UEFI.
2. En el firmware (F.40 en este equipo): desactivar CSM/Legacy, dejar UEFI puro,
   activar TPM 2.0, y **desactivar Secure Boot temporalmente** (se reactiva al final
   con nuestras propias claves vía `sbctl`). Anotar que SGX queda desactivado por BIOS
   (verás `x86/cpu: SGX disabled` en el journal: normal en este HP).
3. Conectar red (cable o `iwctl`) y sincronizar reloj: `timedatectl set-ntp true`.

---

## Parte 1 — Disco: particionado, LUKS2, btrfs y subvolúmenes

> ⚠️ Esto **borra** `/dev/nvme0n1` entero. Comprueba el disco con `lsblk` antes.

### 1.1 Particionado (EFI 1G + resto Linux)

```bash
sgdisk -Z /dev/nvme0n1
sgdisk -n1:0:+1024M -t1:ef00 -c1:EFI -N2 -t2:8304 -c2:LINUXROOT /dev/nvme0n1
partprobe -s /dev/nvme0n1
lsblk /dev/nvme0n1
```

### 1.2 Cifrado LUKS2 (UNA sola vez — el formateo repetido destruye datos)

```bash
cryptsetup luksFormat --type luks2 /dev/nvme0n1p2
cryptsetup luksOpen /dev/nvme0n1p2 linuxroot
```

### 1.3 Sistemas de archivos

```bash
mkfs.vfat -F32 -n EFI /dev/nvme0n1p1
mkfs.btrfs -f -L linuxroot /dev/mapper/linuxroot
```

### 1.4 Subvolúmenes y montaje (punto ESP unificado: `/efi`)

```bash
mount /dev/mapper/linuxroot /mnt
btrfs subvolume create /mnt/home
btrfs subvolume create /mnt/srv
btrfs subvolume create /mnt/var
btrfs subvolume create /mnt/var/log
btrfs subvolume create /mnt/var/cache
btrfs subvolume create /mnt/var/tmp
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

> Nota: las notas originales montaban en `/mnt/efi` pero instalaban el loader con
> `--esp-path=/efi`. Este equipo usa **`/efi`** como definitivo. Usa `/mnt/efi`
> durante la instalación (= `/efi` una vez dentro del chroot).

### 1.5 Espejos y sistema base

Espejos mediante el temporizador de reflector (método real de este equipo: servicio
bajo demanda + `reflector.timer` activo). La configuración vive en
`/etc/xdg/reflector/reflector.conf`:

```ini
--save /etc/pacman.d/mirrorlist
--protocol https
--latest 5
--sort age
--country DE
```

> País **DE** a propósito: residencia en Alemania. Activar con
> `systemctl enable reflector.timer` (el servicio `reflector.service` queda a demanda).
> Las notas originales usaban un comando único con `--country DE --age 24`; el método
> timer es el que quedó instalado y verificado.

```bash
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers \
  intel-ucode btrfs-progs neovim nano sudo networkmanager \
  sbctl tpm2-tools nvidia-dkms cryptsetup dosfstools util-linux git unzip
```

* `nvidia-dkms` (no `nvidia`): compila el módulo contra tu kernel; imprescindible con Secure Boot.
* `linux-headers`: necesario para compilar dicho módulo.

### 1.6 Nota sobre `/etc/fstab` (decisión consciente)

La instalación original **no generó fstab** (`genfstab` no estaba en las notas) y el
sistema arranca por **auto-discovery de systemd** (gpt-auto + LUKS por etiqueta).
Verificado: `/etc/fstab` contiene solo comentarios y el equipo arranca bien.

Se deja así a propósito (funciona y evita UUIDs hardcodeadas), pero que conste:
un `fstab` explícito sería más robusto ante cambios de disco. Si algún día lo quieres:
`genfstab -U /mnt >> /mnt/etc/fstab` **antes** del primer arranque.

---

## Parte 2 — Sistema base dentro del chroot

```bash
arch-chroot /mnt
```

### 2.1 Hora y región

```bash
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
hwclock --systohc
```

> Zona `Europe/Berlin` (residencia en Alemania; verificada con `timedatectl`).
> Las notas originales decían `Europe/Madrid`: corregido.

### 2.2 Locales trilingües de/en/es (versión corregida — los `sed` originales tenían comillas rotas)

Perfil: residente en Alemania, argentino, teclado físico inglés. Se generan los tres
idiomas; `de_DE` como idioma por defecto del sistema (formatos, apps y correctores
del país de residencia), `es_ES` y `en_US` disponibles para el usuario y KDE.

```bash
sed -i -e 's/^#\(de_DE.UTF-8\)/\1/' /etc/locale.gen
sed -i -e 's/^#\(es_ES.UTF-8\)/\1/' /etc/locale.gen
sed -i -e 's/^#\(en_US.UTF-8\)/\1/' /etc/locale.gen
locale-gen
echo "LANG=de_DE.UTF-8" > /etc/locale.conf
echo "KEYMAP=us" > /etc/vconsole.conf
```

> Estado real: `locale.gen` tenía activos `de_DE`, `en_GB`, `es_ES` y `locale.conf`
> quedó en `LANG="C.UTF-8"` (fallback). Estándar nuevo: sustituir `en_GB` por `en_US`
> (teclado físico inglés → `en_US` es lo estándar), `LANG=de_DE.UTF-8` y `KEYMAP=us`
> en consola. En gráfico (KDE/Wayland) se usan layouts conmutables `de,es,us`
> (ya configurados así en este equipo: `~/.config/kxkbrc` → `LayoutList=de,es,us`).

### 2.3 Red

```bash
echo "archlinux" > /etc/hostname
```

```ini
# /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
```

```bash
systemctl enable NetworkManager
systemctl mask systemd-networkd
systemctl enable systemd-resolved systemd-timesyncd sshd
```

> El doc original proponía `HP-Arch-Station`; el equipo quedó como `archlinux`. Se respeta.

### 2.4 Usuarios y sudo (versión segura — sin NOPASSWD)

```bash
passwd                                   # contraseña de root
useradd -m -G wheel -s /bin/bash renato
passwd renato
EDITOR=nano visudo                       # descomentar: %wheel ALL=(ALL:ALL) ALL
```

> ⚠️ Las notas originales activaban `%wheel NOPASSWD: ALL` (sudo sin contraseña).
> **No usar.** La forma correcta es la de arriba: sudo pide contraseña.

---

## Parte 3 — Kernel unificado (UKI), Secure Boot y TPM2

### 3.1 Línea de comandos del kernel (estado real verificado)

```ini
# /etc/kernel/cmdline
quiet rw nvidia_drm.modeset=1 threadirqs
```

### 3.2 mkinitcpio (estado real verificado — SIN hook `resume`)

```ini
# /etc/mkinitcpio.conf
MODULES=(btrfs nvidia nvidia_modeset nvidia_uvm nvidia_drm)
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole sd-encrypt block filesystems fsck)
```

> Diferencias con las notas originales (corregidas aquí):
> * Sin `resume`: no había swap en la instalación, la hibernación era imposible.
>   (La swap zram de 16G se añadió meses después; `resume` sobre zram tampoco aplica.)
> * Sin `fsck`: btrfs no usa fsck al arranque.
> * Con `microcode` y `kms`: añadidos después (ucode Intel y modesetting temprano NVIDIA).

Regenerar UKIs tras cualquier cambio:

```bash
mkinitcpio -P
ls -l /efi/EFI/Linux   # arch-linux.efi + arch-linux-fallback.efi
```

### 3.3 Bootloader

```bash
bootctl install --esp-path=/efi
```

Sin entradas en `loader/entries`: arranque directo por UKI (discovery automático). Normal.

### 3.4 Secure Boot con claves propias (sbctl)

```bash
sbctl status
sbctl create-keys
sbctl enroll-keys -m
sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed \
  /usr/lib/systemd/boot/efi/systemd-bootx64.efi
sbctl sign -s /efi/EFI/BOOT/BOOTX64.EFI
sbctl sign -s /efi/EFI/Linux/arch-linux.efi
sbctl sign -s /efi/EFI/Linux/arch-linux-fallback.efi
```

Tras cada actualización de kernel/systemd-boot, **re-firmar** y comprobar:

```bash
sbctl verify
```

Estado verificado 2026-09-09: las 4 imágenes de arranque firmadas ✓.
Nota menor: `/efi/EFI/systemd/systemd-bootx64.efi` (copia de reserva) aparece sin firmar —
pendiente cosmético, no afecta al arranque.

### 3.5 TPM2: desbloqueo sin contraseña + recovery key

```bash
systemd-cryptenroll /dev/nvme0n1p2 --recovery-key
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 /dev/nvme0n1p2
```

> 🧰 **Troubleshooting real (conservar):** tras el primer enroll con PCRs `0+7` el
> equipo **seguía pidiendo contraseña**. Solución aplicada:
>
> ```bash
> systemd-cryptenroll --wipe-slot=tpm2 --tpm2-device=auto --tpm2-pcrs=7 /dev/nvme0n1p2
> ```
>
> Con solo PCR 7 el desbloqueo automático funcionó. Estado de slots verificado:
> `0 password · 1 recovery · 3 tpm2` (el slot 2 fue el enroll inicial eliminado).

### 3.6 Salir y primer arranque

```bash
sync
umount -R /mnt
reboot
```

Reactivar Secure Boot en firmware tras comprobar el primer arranque firmado.

---

## Apéndice A — Diferencias notas originales vs realidad (todas verificadas)

| Tema | Notas originales | Realidad |
|---|---|---|
| ESP | `/mnt/efi` vs `--esp-path=/efi` mezclados | `/efi` definitivo |
| fstab | sin `genfstab` | vacío, arranque por auto-discovery (voluntario) |
| sudo | `NOPASSWD: ALL` | **corregido:** con contraseña |
| HOOKS | `... sd-encrypt resume filesystems fsck` | sin `resume`/`fsck`, con `microcode`/`kms` |
| locales | `sed` con comillas rotas | reescritos y funcionales |
| LUKS | sección duplicada (peligro `luksFormat` ×2) | una sola ejecución |
| hostname | `HP-Arch-Station` | `archlinux` |
| LANG | `es_ES.UTF-8` | estándar nuevo: `de_DE.UTF-8` (realidad: `C.UTF-8` fallback) |
| KEYMAP consola | `es` | `us` (teclado físico inglés) |
| teclado gráfico | `es` fijo | layouts KDE `de,es,us` conmutables |
| timezone | `Europe/Madrid` | `Europe/Berlin` (residencia DE) |
| locales generados | `de_DE, en_GB, es_ES` | `de_DE, es_ES, en_US` (`en_US` sustituye a `en_GB`) |
| cmdline | `quiet rw nvidia_drm.modeset=1` | + `threadirqs` |
| TPM2 PCRs | `0+7` | `7` (tras fix documentado) |
| reflector | comando único `--country DE --age 24` | método timer, `--country DE` (residencia DE, definitivo) |

## Apéndice B — Verificaciones rápidas post-instalación

```bash
bootctl status | head          # Secure Boot: enabled (user), TPM2: yes
sbctl status && sudo sbctl verify
sudo systemd-cryptenroll /dev/nvme0n1p2   # slots: password + recovery + tpm2
ls /efi/EFI/Linux             # arch-linux.efi + fallback
cat /etc/kernel/cmdline
grep -E '^(HOOKS|MODULES)' /etc/mkinitcpio.conf
timedatectl | grep "Time zone"            # Europe/Berlin
grep -v "^#" /etc/locale.gen | grep -v "^$"  # de_DE, es_ES, en_US
localectl                     # LANG=de_DE.UTF-8, VC Keymap: us
systemctl is-enabled reflector.timer     # enabled
```

## Apéndice C — Archivos originales

* `notas-originales-part1.md`: notas crudas tal cual se tomaron durante la instalación
  (con typos y duplicados preservados como testimonio).
* `Readmi.md` + `Prompts.md`: parte 2 redactada y prompt/tutorial de referencia.
* `originales/`: `code.txt` (duplicado exacto de la parte 2) y `archinstall_part2.md` (stub vacío).
