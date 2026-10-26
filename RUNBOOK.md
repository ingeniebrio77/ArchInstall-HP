# RUNBOOK — Operación diaria sin asistencia

Manual de mantenimiento de este equipo (Arch + Hyprland + estudio de música).
Todo aquí se hace con terminal y vim. Si algo no está en este documento,
`journalctl` y `pw-link -l` dicen la verdad.

## 1. Encendido y apagado (reglas duras)

- **Orden de encendido:** Behringer → io|2 → Axiom → PC. Ardour se abre al final.
- **Antes de apagar el PC:** Axiom OFF (interruptor trasero). Con el Axiom
  encendido por USB, el equipo se queda colgado al apagar (verificado 2 veces).
- **Antes de suspender (`Super+Shift+E`):** VM Windows apagada y SongRec
  cerrado. El hook `/etc/systemd/system-sleep/00-audio-vm-guard` veta si no.
- Suspend = s2idle (S3 cuelga en este HP). No tocar `mem_sleep_default`.

## 2. Audio: arquitectura y rescates

```
Apps (Firefox) → easyeffects_sink → [EQ→limiter] → Burr-Brown (Behringer) → Eris
Ardour Master ─────────────────────────────────→ Burr-Brown → Eris
Axiom (USB + DIN→io|2) → pista MIDI → sinte ACE → Master
```

- `Super+Ctrl+A` — **Rescate**: restaura el estado sano completo (idempotente).
- `Super+Shift+R` — Re-enlaza salida EasyEffects al dispositivo elegido.
- `Super+R` — SongRec directa/ambiente (re-impone cables y quita parásitos).
- `Super+Ctrl+R` — Matar SongRec. Un timer systemd lo re-cablea solo cada 30 s
  si la GUI se engancha al micro (`systemctl --user status songrec-watch.timer`).
- `Super+A` / scroll en waybar — Cambiar dispositivo de salida.
- EasyEffects: los nodos `ee_soe_*` solo existen con audio sonando (no es bug).
  Presets con `Super+O`. La entrada de EE se cambia desde su GUI, nunca a mano.
- Behringer: Eris en MAIN OUT, MAIN MIX arriba, `2-TR/USB TO MAIN` pulsado.
  Si el sistema entero enmudece (ni Firefox): desenchufar/re-enchufar el USB
  de la Behringer (se cuelga aceptando writes sin arrancar el stream).
- Diagnóstico: `pw-link -l` (cables), `wpctl status` (volúmenes/mutes),
  `pw-top` (flujos). Micro de prueba en la Behringer aísla PC vs hardware.

## 3. Ardour + Axiom 49 MK-I

- Sesión/plantilla propia: al crear proyecto, elegir plantilla o clonar con
  `Projekt → Speichern unter…`. **Ctrl+S** fija cables y superficie.
- Pista de teclado = **Midispuren** + instrumento ACE (una pista de audio no
  tiene midi_in: error clásico).
- Cable del teclado: `Fenster → Audioverbindungen → MIDI`: fuente
  `USB Axiom 49: MIDI 1` → destino `AXIOM49/midi_in 1` → Verbinden.
- Cabecera de pista: `In` naranja, sin Mute, fader en 0 (nunca `-inf`).
- Superficie: Einstellungen → Eingabegeräte → Controller → Axiom 49 MK-I,
  Incoming = USB Axiom. Si un control muere: desmarcar/marcar (recarga mapa).
- Mapa propio: `~/.config/ardour9/midi_maps/M-Audio_Axiom49_MKI.map`
  (faders CC74…, master CC7+CC15, transporte CC20-25 y CC80-89, encoders=pan).
- Teclado: Glob Chan 1, Group A (LCD manda). Reset: apagar + mantener **+/-**
  de octava al encender. Pads = canal 10 (batería GM, no suenan con ACE).
- Si Ardour no ve el teclado al abrir: el Axiom estaba apagado al arrancar la
  sesión → reconectar a mano o reabrir sesión.
- Tutoriales completos: `~/ardour-axiom-docs/docs/01..04`.

## 4. Monitores

- LG ultrawide = HDMI-A-1 (WS 1-5) · Toshiba TV = DP-1 (WS 6-10, arranca OFF).
- `Super+X` activa/apaga Toshiba (lee estado real, responde a la primera).
- Al iniciar sesión, `monitor-restore.sh` repone el último estado (exec-once).
- Wallpapers por monitor: `~/.config/hypr/hyprpaper.conf` (Japón: Fuji + Kioto).

## 5. Actualizaciones y Secure Boot

- `sudo pacman -Syu` con regularidad. Huérfanos: `pacman -Qdtq` (revisar a
  mano, nunca comodines a ciegas).
- **Si cayó kernel o mkinitcpio:** las UKIs de `/efi` se regeneran solas y
  quedan SIN firma → re-firmar antes de reiniciar:
  ```
  sudo sbctl verify
  sudo sbctl sign -s /efi/EFI/Linux/arch-linux.efi
  sudo sbctl sign -s /efi/EFI/Linux/arch-linux-fallback.efi
  ```
- Sin `/etc/fstab`: el arranque es auto-discovery de systemd. No crearlo.

## 6. Rollback y snapshots

- snapper + snap-pac: snapshot automático pre/post cada pacman/yay.
- Recuperar un archivo: copiarlo desde `/.snapshots/<n>/snapshot/…` o
  `/home/.snapshots/<n>/snapshot/…`. No hay "arrancar desde snapshot".
- Snapshot manual antes de tocar /etc o configs críticas:
  `sudo snapper -c root create -d "motivo"` / `-c home`.

## 7. KDE adelgazado (2026-10-20)

- Se quitó el escritorio Plasma; quedan: sddm (login), kdeconnect, kdenlive,
  breeze-icons, kwallet (dep. de kdeconnect). Gestor de archivos = ranger
  (`Super+M`). Polkit = hyprpolkitagent · Bloqueo = hyprlock (`Super+Alt+L`).
- Protegidos como explícitos (no dejar que un `-Rs` futuro los barra):
  `ddcutil` (brillo Super+F2/F3), `mpv`, `noto-fonts-emoji`, `xdg-user-dirs`.

## 8. Atajos

`Super+/` muestra la ayuda viva (`~/.config/hypr/atajos.md`).

## 9. Juegos sin flatpak (2026-10-20)

- **Flatpak eliminado** del sistema (apps, runtimes nvidia duplicados, remote
  flathub y el paquete). La GPU no depende de flatpak: el driver del host
  (`nvidia-open-dkms` + `lib32-nvidia-utils`) alimenta todo lo nativo.
- **Steam nativo** (`steam` de multilib). Librería migrada con rsync a
  `~/.local/share/Steam` (9 manifests, 11 juegos, compatdata y shadercache
  íntegros; `libraryfolders.vdf` corregido a la nueva ruta).
- **Ryubing (Ryujinx) nativo** en `/opt/ryujinx/bin` (.desktop e ícono en
  `~/.local/share/`). Datos en `~/.config/Ryujinx` (mismo nombre que usa el
  nativo): `sdcard` = SD virtual con juegos (27G), `bis` = NAND y saves, keys
  incluidas. Actualizar: cuando `git.ryujinx.app` vuelva, instalar
  `ryujinx-bin` de AUR.
- **Mullvad Browser** nativo (`mullvad-browser-bin` de AUR), perfil nuevo.
- **Disco y snapshots (btrfs):** borrar datos no libera espacio hasta que el
  snapshot que fija los bloques viejos se borra o caduca. Tras verificar que
  Steam y Ryubing exponen todo, borrar el snapshot home #171 ("Antes de
  migrar") devuelve ~165G libres: `sudo snapper -c home delete 171`.

## 10. Suite KDE applications — fuera (2026-10-20 tarde)

- Se eliminó el grupo `kde-applications` completo (~174 paquetes: kate,
  konqueror, k3b, kmail/kontact, calligra, marble, 43 juegos, PIM akonadi...),
  más 6 zánganos posteriores y ~141 huérfanos (hasta el `mariadb` embebido de
  akonadi cayó). `pacman -Qdtq` = 0.
- Del grupo quedan solo `kdeconnect`, `kdenlive` y
  `signon-kwallet-extension` (pinado por kdeconnect).
- Fuera duplicados: `alacritty` (vives kitty) e `yazi` (vives ranger).
- Visores nuevos vim-style: `zathura` + `zathura-pdf-mupdf` (PDF)
  e `imv` (imágenes).
- Datos de usuario de apps muertas borrados con snapshot home previo:
  `~/.local/share/akonadi` (117M) y ~30 rc de kate/konsole/kmail/kwin/kmix/
  konqueror. Conservados a propósito: `kdeglobals`, `kdeconnect…`,
  `kdenlive…`, `kwallet…` y `~/.config/containers` (podman).

## 11. Correo y calendario en terminal (aerc + khal, 2026-10-26)

- **aerc** (Super+G) = cliente de correo vim-native: ver manual con `man
  aerc-keybinds` (hjkl, `/`, `n/N`, `:q`, `:m`, `:t`). Cuentas: Disroot +
  Gmail en `~/.config/aerc/accounts.conf`; claves en `~/.local/state/aerc-cred/`
  (0600) creadas con `~/.config/aerc/setup-claves.sh` (read -s, nunca por
  chat). Gmail lleva **app password** (necesita 2FA):
  myaccount.google.com → Seguridad → Contraseñas de app. HTML → filtro
  `~/.config/aerc/filters/html` (links -dump con fallback python).
- **khal** (Super+K) = calendario vim: `hjkl` navegan, `?` ayuda. Lee
  `~/.calendars/*.ics` que baja **vdirsyncer** desde Google CalDAV. Token
  OAuth (solo ese camino acepta Google en CalDAV) generado UNA vez por
  proyecto propio en console.cloud.google.com (habilitar API **CalDAV**,
  no "Calendar") con `~/.config/vdirsyncer/setup-google.sh` (read -s) y
  primera `vdirsyncer sync` (abre URL, autorizas, tokens en
  `~/.local/state/vdirsyncer/`).
- Timer `vdirsyncer.timer` sincroniza cada 30 min (systemd --user). Hasta
  la primera sync autorizada, falla inofensivo — el waybar muestra "libre".
- Waybar: módulo `custom/calendar` (`agenda.sh`): primer evento + tooltip
  de 3 días; click abre khal.
- Thunderbird: sin autostart ni atajo; desinstálalo cuando domines aerc
  (`sudo pacman -Rns thunderbird`).
