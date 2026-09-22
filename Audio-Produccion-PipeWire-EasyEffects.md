# Audio producción en Arch (PipeWire + EasyEffects + 2 interfaces)

> **Para quién:** músicos principiantes en Linux que usan Arch para producir.
> **Idea en 1 frase:** todas las apps suenan por `easyeffects_sink`, EasyEffects corrige audífonos/monitores, y de ahí sale a tu interfaz física (Alesis o Behringer).
> **Probado en:** Arch 2026, PipeWire 1.6.8, WirePlumber 0.5.17, EasyEffects 8.2.9, qpwgraph 1.0.4.

## 1. Tu equipo (lo verificado hoy con `wpctl status`)

- **Alesis iO2 (siempre):** aparece como `io|2 Analoges Stereo` (chip Burr-Brown). 2 entradas. Uso con Beyerdynamic DT 990 Pro 80Ω.
- **Behringer QX1204 (no siempre encendida):** aparece como `PCM2902 Audio Codec Analoges Stereo`. Por USB son solo 2 canales (el main mix). Uso con PreSonus Eris E3.5 1ª gen por MAIN/CTRL ROOM balanceado.
- **HDMI NVIDIA TU116 (LG Ultrawide):** excluido del flujo musical, solo video/sonido escritorio.
- Cadena viva hoy: `Firefox output_FL/FR > Easy Effects Sink:playback_FL/FR`, default sink `easyeffects_sink`, default source `alsa_input.usb-Burr-Brown...analog-stereo-input`.

## 2. Latencia baja: quantum 256 (verificado)

`~/.config/hypr/audio-production-setup.sh` (autostart Hyprland):

```bash
sleep 2
pw-metadata -n settings 0 clock.quantum 256
pw-metadata -n settings 0 clock.min-quantum 32
pw-metadata -n settings 0 clock.max-quantum 512
pw-metadata -n settings 0 clock.allowed-rates '[ 44100, 48000, 88200, 96000 ]'
```

Verificado hoy con `pw-metadata -n settings`: `clock.quantum 256`, `clock.rate 48000`, `allowed-rates [44100,48000,88200,96000]`. 256/48000 = ~5.3 ms por periodo, equilibrio entre latencia y estabilidad USB.

## 3. WirePlumber: no-suspend + quantum por interfaz (verificado)

En `/etc/wireplumber/main.lua.d/` (son de root, toca con snapshot antes):

- `90-no-suspend.lua`: a todo USB `session.suspend-timeout-seconds=0` + `node.pause-on-idle=false`. Evita clics al despertar.
- `91-alesis-i2.lua`: match `device.name == io|2`, `period-size 256`, `period-num 2`, `headroom 256`, no-suspend.
- `92-behringer-qx1204.lua`: match `device.name == USB Audio CODEC` (así se ve la QX1204), mismos 256/2/256 + no-suspend.

## 4. EasyEffects en modo servicio (verificado)

Autostart en `~/.config/hypr/hyprland.conf`:

```
exec-once = easyeffects --service-mode
exec-once = ~/.config/hypr/audio-production-setup.sh
exec-once = ~/.config/hypr/relink-audio.sh
exec-once = ~/.config/hypr/audio-watch.sh
```

- No hay user-service systemd (`systemctl --user is-enabled easyeffects` = not-found, normal). Corre como cliente PipeWire (pid visto en `wpctl status`: `easyeffects` + filtros `easyeffects_sink`, `ee_soe_output_level`, `ee_soe_spectrum`).
- Cadena salida: `app → easyeffects_sink → [equalizer → limiter#0] → ee_soe_output_level → interfaz física`.
- `StreamOutputs/plugins=equalizer#0,limiter#0` (visto en `~/.config/easyeffects/db/easyeffectsrc`).

### Presets (verificados en `~/.local/share/easyeffects/output/`)

- `DT 990 Pro Black Edition 80 Ohm EQ+Limiter.json`: preamp `input-gain -6.0`, 10 bandas, + `limiter#0` (attack 5.0, release 50.0). Para Alesis + DT990.
- `PreSonus Eris E3.5.json`: preamp `-4.3`, 9 bandas tipo Bell RLC (BT) estilo Spinorama Erin's Audio Corner, + `limiter#0` (attack 5.0, release 50.0). Para Behringer + Eris. Es el `lastLoadedOutputPreset` actual.
- Hay variantes `DT 990 ... EQ.json`, `... .json` sin limiter para comparar.

> Discrepancia honesta: `~/.config/easyeffects/autoload/` está vacío hoy. No hay autoload automático por dispositivo activo; la selección es manual (mostUsed: Eris 9, DT990 3, Perfect EQ 7). Si quieres autoload real Alesis→DT990 / Behringer→Eris, hay que crearlo y probarlo en sesión real (tu pendiente).

## 5. Scripts Hyprland (qué hace cada uno)

- `relink-audio.sh`: tras arrancar EasyEffects, desconecta `ee_soe_output_level:output_FL/FR` de donde esté y lo conecta al dispositivo guardado en `~/.cache/audio-device` (default `alsa_output.usb-Alesis_io_2-00.analog-stereo`). Hoy el caché apunta a `alsa_output.usb-Burr-Brown...-output` (= Alesis).
- `audio-switch.sh [next|prev]`: lista sinks físicos con `pw-dump` + python (excluye `easyeffects` y `hdmi`), rota y guarda en `~/.cache/audio-device`. Uso con scroll Waybar o `Super+A` según tu config.
- `audio-watch.sh`: cada 5 s compara sinks, si aparece uno nuevo (ej. enciendes la QX1204) manda `notify-send "Nuevo dispositivo: Behringer/Alesis"`.
- `vol.sh up|down|mute`: volumen siempre sobre `easyeffects_sink` con tope 100% (evita saturar el limiter).
- `toggle-bypass.sh [on|off|toggle]`: `easyeffects -b 1` = bypass (señal limpia), `-b 2` = efectos. Guarda estado en `/tmp/easyeffects-bypass` y notifica. No toca routing, ambas salidas siguen conectadas.
- Binds: `Super+Shift+B` = bypass toggle, `Super+U` = abrir EasyEffects.

## 6. Herramientas gráficas

- `qpwgraph 1.0.4-1` conservado para ver/cablear nodos.
- `jack_delay` conservado para medir latencia real.
- `helvum` y `cable` desinstalados (`which` no los encuentra, a propósito KISS).

## 7. Verificación en sesión real (tu pendiente)

```bash
wpctl status                       # nodos io|2, PCM2902, easyeffects_sink existen y no Muted
pw-metadata -n settings            # quantum 256, allowed-rates [...]
pw-top                             # carga DSP, sin xruns cantados
cat ~/.cache/audio-device          # a dónde sale EasyEffects
journalctl --user -u pipewire -u wireplumber -n 50 --no-pager
```

Pruebas:
1. Hotplug Behringer: enciende QX1204 con sesión abierta, espera aviso `audio-watch.sh`, cambia con `audio-switch.sh next`, confirma que suena Eris con preset Eris.
2. Bypass: `Super+Shift+B`, confirma aviso `BYPASS (señal limpia)` / `NORMAL`, compara DT990 con/sin EQ en tema conocido.

## 8. Copia de seguridad (qué guardar)

No hay nada en `/etc` que hayas inventado salvo las 3 reglas WirePlumber. Guarda:

```bash
cp /etc/wireplumber/main.lua.d/90-no-suspend.lua /etc/wireplumber/main.lua.d/91-alesis-i2.lua /etc/wireplumber/main.lua.d/92-behringer-qx1204.lua ~/backup-audio/
cp -r ~/.config/hypr/audio-*.sh ~/.config/hypr/toggle-bypass.sh ~/.config/hypr/vol.sh ~/backup-audio/
cp -r ~/.local/share/easyeffects/output/ ~/backup-audio/easyeffects-output/
```

Antes de tocar `/etc/wireplumber`: `sudo snapper -c root create -d "Antes de tocar WirePlumber audio"`.
