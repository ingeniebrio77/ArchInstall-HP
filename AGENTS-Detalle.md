# AGENTS-Detalle.md — Notas narrativas del sistema (consulta bajo demanda)

Contenido movido fuera de `~/AGENTS.md` (2026-10-28) para reducir tokens de contexto.
`AGENTS.md` conserva solo las reglas operativas; aquí está el historial y el porqué.
Consúltalo cuando trabajes en audio, suspensión, VM o NVIDIA.

## Kernel / suspensión

- Cmdline: `quiet rw nvidia_drm.modeset=1 threadirqs mem_sleep_default=s2idle`.
- s2idle desde 2026-09-28: S3/`deep` cuelga el arranque intermitentemente en este HP incluso sin carga; s2idle probado limpio.
- Historial de cuelgues de suspensión:
  - **2026-09-23:** con VM Win11 corriendo + passthrough USB, cuelgue en S3 (`PM: suspend entry` sin `exit`, apagado a botón). Sin VM, el suspend funciona. Regla: apagar Windows desde Inicio antes de suspender el host.
  - **2026-09-24:** sin VM pero con SongRec capturando + cadena de entrada EE activa también colgó; con audio en reposo funciona limpio (entry 10:55 → exit 10:57 sin errores xhci). Regla: cerrar SongRec y toda captura antes de suspender.
  - **AUTOMATIZADO 2026-09-24** en `/etc/systemd/system-sleep/00-audio-vm-guard` (pre: veta con exit 1 + aviso si la VM corre, cierra SongRec; post: `dpms on` + relink-audio contra pantalla negra y reenumeración USB; verificado todo).
  - **2026-09-28 02:17:** tercer cuelgue SIN VM ni audio activos (kernel colgado en `suspend entry (deep)` puro = firmware HP): solución adoptada `mem_sleep_default=s2idle` en cmdline (ciclo s2idle verificado limpio).

## Audio (PipeWire / WirePlumber / EasyEffects)

- PipeWire quantum 256 ≈ 5.3 ms, rates `[44100,48000,88200,96000]` vía `~/.config/hypr/audio-production-setup.sh` (autostart Hyprland). WirePlumber: `90-no-suspend`, `91-alesis`, `92-behringer` en `/etc/wireplumber/main.lua.d/`.
- Interfaces: Alesis iO2 (siempre, + DT 990 Pro 80Ω) y Behringer QX1204 PCM2902 (no siempre encendida, solo 2 canales por USB, + PreSonus Eris E3.5 por MAIN/CTRL ROOM balanceado).
- EasyEffects `--service-mode`, cadena `app → easyeffects_sink → [EQ → limiter]`; presets `DT 990 Pro Black Edition 80 Ohm EQ+Limiter` (preamp −6.0 dB, 10 bandas) y `PreSonus Eris E3.5` (9 bandas Spinorama, preamp −4.3 dB). Bypass `Super+Shift+B` (`toggle-bypass.sh`, no toca routing). `qpwgraph` y `jack_delay` conservados; `helvum` y `cable` desinstalados.
- Scripts de audio en `~/.config/hypr/`: `audio-production-setup.sh`, `relink-audio.sh`, `audio-switch.sh`, `audio-watch.sh`, `vol.sh`, `toggle-bypass.sh`, `songrec-source.sh` (SongRec directa/ambiente, `Super+R`), `ee-preset.sh` (Eris/DT990, `Super+O`; OJO: `easyeffects -l` solo aplica una vez por proceso, el script escribe `lastLoadedOutputPreset` y reinicia el servicio+relink, ~12 s sin audio).

### Lecciones EasyEffects (2026-09-23)

- Los nodos de efectos (`ee_soe_equalizer`, `ee_soe_limiter`, `ee_sie_*`) solo existen durante reproducción/captura activa (ahorro CPU de EE): verificar siempre con audio sonando; en silencio es normal no verlos.
- El preset Eris/DT990 traía `limiter release=50` fuera de rango (máx 20, aviso `setRelease` en journal): clampado a 20.0 en ambos JSON.
- EE rechaza `alsa_input.usb-Burr-Brown...` escrito a mano en el config (revierte a `easyeffects_sink`): cambiarlo desde la GUI (PipeWire → desactivar `Standardeingang nutzen` → Name = PCM2902), verificado con voz real por speex+autogain.

### QX1204 (verificado 2026-09-23)

- El USB lleva **solo el MAIN MIX**. Monitorear por CTRL ROOM con el MAIN abajo = parlantes suenan pero USB mudo. Regla: Eris en MAIN OUT, canal asignado a MAIN, fader MAIN MIX arriba.
- SM58 (dinámico): phantom OFF, TRIM generoso (picos de voz ~−12 dBFS).

## VM Windows 11 (BOSS RC-5 Tone Studio)

- El TPM/Secure Boot del host no se toca (la VM usa `swtpm-win11.service` + OVMF en `~/secboot/`).
- En el instalador: driver de disco `viostor` y de red `NetKVM/w11/amd64` desde `virtio-win.iso`; **NO tocar ni formatear el disco `BOSS RC-5 (D:)`**.
- El PID USB del RC-5 cambia por modo (visto `0582:0251`): verificar con `lsusb -d 0582:` antes de usar `--boss`.
- `qemu-img: Failed to get shared "write" lock` significa VM corriendo, no error.
- Cursor: `-device usb-tablet` obligatoria (coordenadas absolutas; sin ella el mouse va a saltos con `-vga virtio`); más driver `viogpu` de las guest-tools.
- Backup del `qcow2` solo con VM apagada (`cp --sparse=always`).
- Resto viejo `~/win11-vm/` (disco 22G de abril) no es el canónico: no borrar hasta backup verificado.

## Wayland / NVIDIA

- `nvidia-dkms` con parámetros tempranos de kernel y variables de entorno específicas para Hyprland. Ante fallos gráficos o glitches, revisar que `WLR_NO_HARDWARE_CURSORS`, `LIBVA_DRIVER_NAME`, etc. no colisionen con las actualizaciones recientes del driver.

## Teclado Cherry (2026-10-28)

- El Cherry `cherry-wired-keyboard` (USB `046a:007d`) solo envía F1–F12 (no tiene teclas de consumo `XF86Audio*` ni brillo en su descriptor HID): por eso F1–F12 (+ alias F13–F24) están mapeadas a multimedia en `hyprland.conf` (brillo F1/F2, volumen F3–F5, player F6–F8, mic F9, nocturno F10, captura F11, bloqueo F12).
- Layout por dispositivo: Cherry `de,us,es` (arranca en DE); el Keychron conserva el global `us,es,de`.
- Waybar: `kb.sh` muestra el layout del teclado principal y rota con clic; `caps.sh` muestra el Bloq Mayús del teclado principal (antes ambos eran Keychron-only).
