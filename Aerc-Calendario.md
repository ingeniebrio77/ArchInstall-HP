# aerc + calendario — cheatsheet (2026-10-27)

## Arranque y credenciales
- `Super+G` → kitty + aerc con `TERM=xterm-256color` (workaround al cuelgue de
  Vaxis con `xterm-kitty`: ~60 s sin pintar y teclado muerto; verificado 26-10).
- Desde fish, la función `aerc` (`~/.config/fish/conf.d/aerc.fish`) ya fija ese TERM.
- Claves en `~/.local/state/aerc-cred/` (0600). Gestionar SIEMPRE con
  `~/.config/aerc/pon-claves.sh [gmail|disroot]` (valida login IMAP antes de guardar).
- Cuentas: Gmail operativa. Disroot sin credencial → diálogo de error al arrancar
  (se cierra con Enter); reintentar con `pon-claves.sh disroot` cuando se quiera.

## Lista de mensajes
| Tecla | Acción |
|---|---|
| `j` / `k` | abajo / arriba |
| `C-d` / `C-u` | media página abajo / arriba |
| `C-f` / `C-b`, `PgDn` / `PgUp` | página completa |
| `g` / `G` | primer / último mail |
| `Enter` | abrir en visor |
| `J` / `K` (`C-Down`/`C-Up`) | carpeta siguiente / anterior |
| `H` / `L` | plegar / desplegar carpeta; `tf` toggle |
| `v` / `Space` / `V` | marcar / marcar+avanzar / marcar visual |
| `d` / `D` | borrar con confirmación / directo |
| `a` / `A` | archivar / archivar marcados |
| `C` o `m` | redactar |
| `rr`/`rq`, `Rr`/`Rq` | responder a todos / solo remitente (q = con cita) |
| `c` | cambiar carpeta (`:cf`) |
| `/` buscar · `n`/`N` resultados · `\` filtro · `Esc` limpiar | |
| `s` / `S` | split horizontal / vertical |
| `$` o `!` | terminal embebido; `|` pipe a comando |

## Visor
| Tecla | Acción |
|---|---|
| `q` | cerrar y volver a la lista |
| `H` | mostrar/ocultar cabeceras |
| `Tab` / `Shift+Tab` | parte siguiente / anterior (también `C-j`/`C-k`) |
| `O` / `o` | abrir adjunto con manejador del sistema |
| `S` | guardar adjunto a archivo |
| `|` | pipe de la parte a un comando |
| `C-y` / `C-l` | copiar link / abrir link |
| `f`, `rr`/`rq`/`Rr`/`Rq` | reenviar / responder |

- Filtros activos (`aerc.conf [filters]`): `text/plain = cat`,
  `text/html = ~/.config/aerc/filters/html` (links -dump),
  `text/calendar = calendar` (parsea invitaciones).
- `image/*` **sin filtro a propósito**: Vaxis las pinta inline con el protocolo
  gráfico de kitty (fallback half-blocks).

## Redacción
- `C-j`/`C-k` o `Tab` cambian de campo; editor = nvim.
- Pantalla de revisión: `y` enviar · `n` descartar · `p` posponer · `e` editar ·
  `a` adjuntar · `d` quitar adjunto · `v` preview.
- Firma: solo Gmail, vía `signature-file` en `accounts.conf`
  (`~/.config/aerc/signature.txt`; aerc añade el separador `-- ` solo).

## Globales
`C-n`/`C-p` pestañas · `C-t` terminal · `?` ayuda de teclas · `C-c`/`C-q` salir con confirmación.

## Calendario (khal/ikhal + vdirsyncer)
- Click en 📅 de waybar → `kitty -e ikhal`.
- ikhal: flechas/space/backspace moverse · `Tab`/`Enter` cambiar de columna ·
  `t` hoy · `n` evento nuevo · `v` marcar rango (`o` otro extremo, `Esc` sale) · `/` buscar.
- CLI: `khal list today 3d` (agenda; en khal 0.14 el subcomando `agenda` ya no
  existe), `khal calendar`, `khal new START [DUR] "Título"`, `khal search ...`.
- Colección local (default para eventos nuevos): `~/.local/share/khal/calendars/local`.
- Google: vdirsyncer sincroniza `~/.calendars/*` (khal los descubre con `type = discover`).
  Timer de usuario `vdirsyncer.timer` = sync cada 10 min; manual: `vdirsyncer sync`.
- Primera vez / re-autorizar: `~/.config/vdirsyncer/setup-google.sh` (OAuth client_id/secret)
  y marcar calendarios en https://calendar.google.com/calendar/syncselect.
- Waybar muestra la agenda de hoy (refresco 5 min; tooltip = 3 días).

## Pendientes conocidos
- Rotada la app password de Gmail el 27-10 → re-guardar con `pon-claves.sh gmail`.
- Sync Google Calendar: pendiente de pasos OAuth en Cloud Console (ver sección calendario).
- Tareas TODO puras: ikhal gestiona eventos; si se quieren TODOs, `todoman` (mismo ecosistema vdir).
