# Atajos de teclado (Hyprland, tecla principal = Super)

> Fuente: `~/.config/hypr/hyprland.conf`. Ver todos en vivo con `Super+/`.

## Dónde vive cada atajo (mapa)

Cada programa guarda SUS teclas — saber dónde buscar es medio trabajo:

| Capa | Archivo | Contiene |
|---|---|---|
| Hyprland (compositor) | `~/.config/hypr/hyprland.conf` | todos los `Super+…` (apps, foco, ws, audio, VM, quake, sesiones) |
| Scripts detrás de los binds | `~/.config/hypr/*.sh` | `quake.sh`, `sesion.sh`, `vol.sh`, `ee-preset.sh`, … |
| Correo aerc | `~/.config/aerc/binds.conf` | teclas vim por contexto (`[messages]`, `[compose]`…); `?` dentro de aerc muestra la ayuda del contexto |
| Terminal kitty | `~/.config/kitty/kitty.conf` | sin keybinds propias — todo por defecto de kitty (el archivo es solo tema/colores) |
| Shell fish | `~/.config/fish/conf.d/abbrs.fish` | abreviaturas (`upds`, `syncmail`, `gs`…); se cargan al abrir cada fish; ver con `abbr --show` |
| Funciones fish viejas | `~/.config/fish/functions/` | fzf, z, yt, tide (no editar a mano) |

Consulta en vivo: `hyprctl binds` (Hyprland) · `abbr --show` (fish) · `?` (dentro de aerc).

Criterio de asignación (para atajos futuros):
- Las letras de `Super+letra` están prácticamente agotadas (solo queda `S`, ya usada por el quake): las apps nuevas van a `Super+Shift+letra` o `Super+Ctrl+letra`.
- `Super+Ctrl+X` = rutinas automatizadas (M mail, S sistema, R SongRec, W BOSS).
- `Super+Shift+X` = acciones de pantalla/sistema (S captura, E suspender, B bypass…).

## Ventanas y workspaces

| Atajo | Acción |
|---|---|
| `Super+Return` | Terminal (kitty) |
| `Super+D` | Lanzador (wofi) |
| `Super+M` | Archivos (dolphin) |
| `Super+Q` | Cerrar ventana |
| `Super+F` | Pantalla completa |
| `Super+V` | Ventana flotante |
| `Super+P` | Pseudo-tile (no-op con layout master, era de dwindle) |
| `Super+T` | Intercambiar con la maestra (`swapwithmaster`, layout master) |
| `Super+K` | Calendario interactivo (ikhal) |
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
| `Super+Shift+R` | Reparar audio: re-enlaza salida EE (`relink-audio.sh`) |

## Sesiones kitty y terminal desplegable (ver `Sesiones-kitty-Quake.md`)

| Atajo | Acción |
|---|---|
| `Super+S` | Terminal desplegable quake (Shell): kitty a pantalla en special workspace, abrir/cerrar en el monitor enfocado |
| `Super+Ctrl+M` | Sesión mail: aerc + ikhal en ws2 (no duplica lo que ya corre) |
| `Super+Ctrl+S` | Panel sistema desplegable: btop + journalctl -xf encima de todo (special workspace); 2ª pulsación lo esconde, tercera lo vuelve a mostrar. Los procesos siguen vivos mientras está escondido |

- Scripts: `~/.config/hypr/quake.sh`, `~/.config/hypr/sesion.sh` (con `mail|sys`).
- Abreviaturas fish (se expanden al teclear): `~/.config/fish/conf.d/abbrs.fish`
  → `gs gd gl ga gc gp bra upd upds pdi pori snaps syncmail vol mem disco temps log av vent`.
  Útil: `upds` = `pacman -Syu` + lista de snapshots snapper; `syncmail` = vdirsyncer + vista de agenda.

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
