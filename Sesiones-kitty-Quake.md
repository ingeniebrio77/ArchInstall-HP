# Sesiones kitty por tarea + terminal desplegable (quake)

> Fecha: 2026-10-27 · Scripts en `~/.config/hypr/`

## 1. Terminal desplegable tipo quake (`quake.sh`)

Kitty con `--class quake` vive en un **special workspace** (`special:quake`).
`Super+S` (S de Shell — única letra libre en `Super+letra`) la abre/cierra en
el monitor enfocado; el contenido persiste (aerc, monitor, lo que tengas
dentro) aunque la escondas en pantalla.

Funcionamiento del script: si no existe un cliente con clase `quake`, lo crea
con `hyprctl dispatch exec '[workspace special:quake silent] kitty --class quake'`,
espera (loop de 0.1 s) y lo muestra con `togglespecialworkspace`.

Verificación usada: `hyprctl clients -j` → clase `quake`, `ws=-98`,
`fullscreen=0` (con kitty 0.48.2 tras el downgrade, ver
`Hyprland-Master-Layout-kitty.md`).

## 2. Sesiones kitty por tarea (`sesion.sh {mail|sys}`)

Abre ventanas kitty (cada una un toplevel → las tila el layout master) en el
workspace que le corresponde, **sin duplicar** apps que ya corren (pgrep):

| Sesión | Dónde sale | Ventanas |
|---|---|---|
| `mail` | ws2 (HDMI, silencioso) | aerc, ikhal |
| `sys` | **popup** `special:sys` (encima de todo, cualquier monitor) | btop, `journalctl -xf` |

Atajos: `Super+Ctrl+M` (mail), `Super+Ctrl+S` (sistema).

La sesión `sys` funciona como el quake: primera pulsación crea las ventanas y
muestra el popup; si ya existen, `Super+Ctrl+S` solo las muestra/esconde
(`togglespecialworkspace sys`). Los procesos btop/journal siguen vivos con el
popup escondido; para matarlos de verdad ciérralos con `Super+Q` dentro.

Duplicación evitada con `pgrep -f` anclado a la ruta del binario
(`(^|/)aerc`, `(^|/)ikhal`...) para no casar con editores u otros procesos que
mencionen el nombre.

## 3. Abreviaturas fish (`~/.config/fish/conf.d/abbrs.fish`)

Se expanden al teclear espacio (abbr de fish — no ensucian el histórico):

- **git**: `gs` status, `gd` diff, `gl` log, `ga` add, `gc` commit, `gp` push,
  `bra` rama actual.
- **paquetes**: `upd` `pacman -Syu`, `upds` `pacman -Syu + snapper list`,
  `pdi` `pacman -S`, `pori` búsqueda local `pacman -Qs`, `snaps`, `resp`
  undochange de snapper.
- **correo/calendario**: `syncmail` = `vdirsyncer sync` + `khal list today 7d`
  (por si el timer de 10 min se quedó corto).
- **monitoreo**: `vol` volumen `wpctl`, `mem` `free -h`, `disco` `df -h`,
  `temps` `sensors`, `log` journal de errores, `av`/`vent` ventanas por
  `hyprctl` (clase/ws/fs).

## Posibles extensores

- Más sesiones: añadir un `case` en `sesion.sh` + un bind.
- Quake arrancando una app en vez de fish: cambiar el spawn a
  `kitty --class quake -e aerc` en `quake.sh` — ojo: si ese proceso muere, la
  ventana muere; la fish interactiva aguanta todo.
