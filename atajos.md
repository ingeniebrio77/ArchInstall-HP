# Atajos de teclado (Hyprland, tecla principal = Super)

> Fuente: `~/.config/hypr/hyprland.conf`. Ver todos en vivo con `Super+/`.

## Ventanas y workspaces

| Atajo | Acción |
|---|---|
| `Super+Return` | Terminal (kitty) |
| `Super+D` | Lanzador (wofi) |
| `Super+M` | Archivos (dolphin) |
| `Super+Q` | Cerrar ventana |
| `Super+F` | Pantalla completa |
| `Super+V` | Ventana flotante |
| `Super+P` | Pseudo-tile |
| `Super+T` | Alternar split |
| `Super+E` | Salir de Hyprland |
| `Super+H/J/K/L` o flechas | Foco izquierda/abajo/arriba/derecha |
| `Super+Shift+H/J/K/L` o flechas | Mover ventana |
| `Super+Ctrl+H/J/K/L` o flechas | Redimensionar |
| `Super+1..9,0` | Ir al workspace 1..10 |
| `Super+Shift+1..9,0` | Mover ventana al workspace 1..10 |
| `Super+[/]` | Workspace anterior/siguiente |
| `Super+Shift+[/]` | Mover a workspace anterior/siguiente |
| `Super+Tab` | Workspace anterior |

## Audio producción (ver `Audio-Produccion-PipeWire-EasyEffects.md`)

| Atajo | Acción |
|---|---|
| Teclas multimedia subir/bajar/mute | Volumen `easyeffects_sink` con tope 100% (`vol.sh`) |
| Teclas play/next/prev | `playerctl` |
| `Super+A` / `Super+Shift+A` | Dispositivo siguiente/anterior (`audio-switch.sh`) |
| `Super+U` | Abrir EasyEffects |
| `Super+Shift+B` | Bypass EasyEffects on/off (`toggle-bypass.sh`, no toca routing) |
| `Super+R` | SongRec: escucha directa ↔ ambiente (`songrec-source.sh`; reabrir GUI tras cambiar) |
| `Super+O` | Preset Eris E3.5 ↔ DT 990 Pro (`ee-preset.sh`, ~12 s sin audio al conmutar) |

## VM Windows 11 (ver `VM-Windows11-ToneStudio-RC5.md`)

| Atajo | Acción |
|---|---|
| `Super+W` | Arrancar Windows 11 |
| `Super+Ctrl+W` | Arrancar Win11 + BOSS RC-5 (`--boss`) |

> `Super+Shift+W` está ocupado por el menú de wallpaper, por eso el modo BOSS usa `Ctrl`.

## Capturas, pantalla y sistema

| Atajo | Acción |
|---|---|
| `Super+Shift+S` | Captura de región |
| `Super+Print` o `Print` o `Fn1+L` (Keychron) | Pantalla completa |
| `Super+Alt+L` | Bloquear pantalla |
| `Super+/` | Ver todos los atajos |
| `Super+Shift+W` | Cambiar wallpaper |
| `Super+-/=`, `Super+F2/F3` o teclas brillo | Brillo del monitor |
| `Super+Z` | Toggle monitor LG Ultrawide |
| `Super+X` | Toggle TV Toshiba |
| `Super+I` | WiFi |
| `Super+B` | Bluetooth |
| `Super+Shift+E` | Suspender (apagar antes la VM Win11) |
| `Super+N` | Filtro nocturno (hyprsunset) |
