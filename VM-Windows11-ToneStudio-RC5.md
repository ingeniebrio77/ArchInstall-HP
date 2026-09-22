# VM Windows 11 para BOSS RC-5 Tone Studio (QEMU en Arch)

> **Para quién:** músicos principiantes en Linux que usan Arch como sistema de producción.
> **Qué resuelve:** BOSS Tone Studio para el RC-5 solo existe en Windows/Mac. En Arch nativo el RC-5 solo da Storage (WAV 44.1 kHz/16 bit) + MIDI. Esta VM es solo editor/librería, tu audio serio sigue en PipeWire + EasyEffects + Alesis/Behringer.
> **Probado en:** HP M01-F1xxx, Arch 2026, QEMU 11.1.1, swtpm 0.10.2, Win11 25H2 EnglishInternational x64.

## 1. Conceptos en 1 minuto (sin jerga)

- **QEMU + KVM:** programa que crea un PC falso dentro de tu Arch. KVM usa tu CPU real para que vaya rápido.
- **qcow2:** un archivo que hace de disco duro (`~/win11_disk.qcow2`). Crece solo según lo usas.
- **OVMF/secboot:** BIOS moderna emulada para que Windows 11 crea que está en un PC real con Secure Boot.
- **swtpm:** chip TPM2 falso por software. Windows 11 lo exige para instalarse. No toca tu TPM real del HP.
- **virtio:** drivers paravirtualizados (disco `viostor`, red `NetKVM`) para que Windows hable rápido con QEMU. Vienen en `virtio-win.iso`.
- **Passthrough USB:** prestar tu pedal físico a la VM con `--boss`.

Tu TPM/Secure Boot real del host (sbctl, PCRs, `/efi`) no se toca en este proceso.

## 2. Qué tienes instalado (estado real verificado 2026-09-22)

```bash
~/win11_disk.qcow2                    # 14G actual, formato qcow2, if=virtio
~/secboot/OVMF_CODE.secboot.4m.fd     # firmware, solo lectura
~/secboot/OVMF_VARS_4M.secboot.fd     # variables NVRAM de la VM
~/tpmstate/                           # swtpm-sock + tpm2-00.permall
~/Downloads/ISOs/Win11_25H2_EnglishInternational_x64_v2.iso  # 8,0G
~/Downloads/ISOs/virtio-win.iso       # 837M, versión 0.1.302
~/.config/hypr/win11-vm.sh            # lanzador
~/.local/share/applications/win11-vm.desktop        # aparece en wofi
~/.local/share/applications/win11-vm-boss.desktop   # con pedal
/etc/udev/rules.d/99-boss-rc5.rules   # permiso USB al grupo kvm
```

> Nota: existe un resto viejo `~/win11-vm/` con otro `win11_disk.qcow2` de 22G de abril. El canónico es `~/win11_disk.qcow2` (el que usa el script). No borres el viejo hasta terminar y hacer backup.

Regla udev verificada:

```
SUBSYSTEM=="usb", ATTR{idVendor}=="0582", MODE="0660", GROUP="kvm"
```

Servicio TPM usuario verificado (`systemctl --user cat swtpm-win11.service`):

```ini
[Unit]
Description=TPM emulado para VM Windows 11 (QEMU)
[Service]
ExecStart=/usr/bin/swtpm socket --tpmstate dir=%h/tpmstate --ctrl type=unixio,path=%h/tpmstate/swtpm-sock --log level=20 --tpm2
Restart=on-failure
[Install]
WantedBy=default.target
```

Usuario verificado en grupos `kvm, libvirt, audio, realtime, wheel` (`id`).

Pedal verificado hoy: `Bus 001 Device 008: ID 0582:0251 Roland Corp. BOSS_RC-5`.

## 3. El lanzador (qué hace cada línea)

`~/.config/hypr/win11-vm.sh`:

```bash
#!/usr/bin/env bash
#   win11-vm.sh        -> arranque normal (disco instalado)
#   win11-vm.sh --boss -> + pedal BOSS RC-5 por USB
#   win11-vm.sh --iso  -> + ISOs de instalacion/drivers
set -e
cd "$HOME"
ISO_WIN="$HOME/Downloads/ISOs/Win11_25H2_EnglishInternational_x64_v2.iso"
ISO_VIRTIO="$HOME/Downloads/ISOs/virtio-win.iso"
# --iso añade 2 cdroms, --boss añade usb-host con vendorid 0x0582 productid 0x0251
systemctl --user start swtpm-win11.service
exec qemu-system-x86_64 \
  -enable-kvm -m 8192 -smp 6 -cpu host -machine q35 \
  -drive if=pflash,format=raw,readonly=on,file=secboot/OVMF_CODE.secboot.4m.fd \
  -drive if=pflash,format=raw,file=secboot/OVMF_VARS_4M.secboot.fd \
  -chardev socket,id=chrtpm,path=tpmstate/swtpm-sock \
  -tpmdev emulator,id=tpm0,chardev=chrtpm \
  -device tpm-tis,tpmdev=tpm0 \
  -drive file=win11_disk.qcow2,format=qcow2,if=virtio \
  -net nic,model=virtio -net user \
  -vga virtio -rtc base=localtime \
  -device qemu-xhci,id=xhci
```

- `-m 8192 -smp 6 -cpu host`: 8G RAM, 6 hilos, CPU del host. Suficiente para Tone Studio.
- `-machine q35`: chipset moderno que Windows 11 espera.
- `-drive ... if=virtio`: disco rápido, por eso Windows pide driver `viostor` al instalar.
- `-net nic,model=virtio`: red rápida, por eso pide driver `NetKVM`.
- `usb-host ... 0582:0251`: solo con `--boss`, le entrega el RC-5.

## 4. Instalación paso a paso (lo que ya hiciste)

### 4.1 Arrancar en modo instalación

```bash
~/.config/hypr/win11-vm.sh --iso
```

Qué hace: arranca con el ISO de Windows + el ISO virtio como 2 lectoras (E: y F: dentro de Windows).

### 4.2 Disco: Load Driver viostor, NO tocar BOSS RC-5

Cuando el instalador no ve el disco: `Load Driver` > buscar en `F:` > `viostor/w11/amd64` > aparece el disco virtio.

> **Regla de oro:** en el selector de discos verás `BOSS RC-5 (D:)`. No formatear, no instalar ahí, no borrar. Es tu pedal en modo Storage.

### 4.3 Red: driver NetKVM

Pantalla `Let's connect you to a network` > `Install driver` > en `F:` busca carpeta `NetKVM`:

```
F:\NetKVM\w11\amd64 -> Select Folder
```

Elige `Red Hat VirtIO Ethernet Adapter` > `Next`. Si ya pone `Network Connected` y `Next` azul (tu caso del 2026-09-22), dale `Next` directo, el driver fino se instala después dentro de Windows.

Por qué: Windows 11 exige red en el OOBE, y la NIC virtio sin driver lo bloquea.

### 4.4 Terminar OOBE (pendiente al documentar)

Cuenta local u online, privacidad, updates. No hay prisa, la VM está sin activar y funciona para Tone Studio.

## 5. Post-instalación (hazlo en este orden)

1. Dentro de Windows, abre `F:` (virtio-win) e instala `virtio-win-guest-tools.exe` (trae `viostor` + `NetKVM` + `guest-agent` + `qemu-ga`). Reinicia.
2. Apaga la VM, arranca normal sin ISOs para comprobar:
   ```bash
   ~/.config/hypr/win11-vm.sh
   ```
3. Instala BOSS Tone Studio for RC-5 (descarga de Roland/BOSS) dentro de la VM.
4. Prueba pedal:
   ```bash
   lsusb -d 0582:   # el PID cambia según modo Storage/MIDI, hoy es 0251
   ~/.config/hypr/win11-vm.sh --boss
   ```
   Si `lsusb` muestra otro PID distinto a `0251`, avisa: hay que ajustar `--boss` en el script.
5. Si el pedal no aparece en la VM: `journalctl --user -u swtpm-win11 -n 50 --no-pager` y revisa el grupo `kvm` (`id`).

## 6. Backup del disco (obligatorio, no hay off-site)

Los snapshots de snapper no cubren fallo físico del NVMe. El `qcow2` es tu Windows entero.

Solo con la VM **apagada**:

```bash
cp --sparse=always ~/win11_disk.qcow2 /ruta/destino/win11_disk_$(date +%F).qcow2
ls -lh /ruta/destino/
```

Verifica arrancando normal una vez tras el backup. Guarda una copia fuera del NVMe (USB externo).

Protección rápida antes de toquetear reglas/host:

```bash
sudo snapper -c home create -d "Antes de tocar VM Win11"
```

## 7. Verificación rápida

```bash
systemctl --user status swtpm-win11.service
ls -lh ~/win11_disk.qcow2 ~/secboot/ ~/tpmstate/ ~/Downloads/ISOs/
cat ~/.config/hypr/win11-vm.sh
cat /etc/udev/rules.d/99-boss-rc5.rules
lsusb -d 0582:
```

## 8. Fallos típicos para principiantes

- `Next` gris en red: falta `NetKVM/w11/amd64`. Rehaz 4.3.
- Instalador no ve disco: falta `viostor`. Rehaz 4.2.
- `qemu-img: Failed to get shared write lock`: normal, la VM está corriendo. Apágala para backup/info.
- Tras regenerar UKIs del host te pide recovery TPM: es del host, no de esta VM. No re-enrolles el swtpm.
- Hotplug Behringer/QX1204: no tiene que ver con esta VM, se prueba en sesión real con `wpctl status`.
