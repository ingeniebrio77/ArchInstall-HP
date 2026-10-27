# Monitores y workspaces: estrategia de dos pantallas

> Fecha: 2026-10-27 · Hyprland 0.56.2 · Barra: waybar (una instancia por output)

## Física y asignación

| Monitor | Conexión | Resolución | Posición | Workspaces |
|---|---|---|---|---|
| LG ultrawide | HDMI-A-1 | 2560x1080 | frente a ti (`0x0`) | **1-5** |
| Toshiba TV | DP-1 | 1920x1080 | a tu izquierda (`-1920x0`) | **6-10** |

## Precedencia de reglas (qué manda, en orden)

1. **Geometría declarada** en `hyprland.conf`: `monitor=DP-1,1920x1080@60,-1920x0,1`
   (ya no está `disable`: Hyprland la enciende solo al detectarla).
2. **Voluntad manual** (cache `~/.cache/monitor-state`): lo que apagaste con
   `Super+Z`/`Super+X` se respeta mientras el monitor esté presente.
3. **Presencia física**: TV apagada = DP-1 no existe; entonces sus ws6-10 viven
   en el monitor restante y `Super+6..0` sigue funcionando (sin verse en la
   barra, decisión tomada 2026-10-27).

## El cerebro único: `monitor-policy.py` + `monitor-apply.sh`

`monitor-policy.py` decide (idempotente) y `monitor-apply.sh` ejecuta:
- geometría correcta por estado (LG frontal; Toshiba a la izquierda salvo que
  sea el único → pasa a `0x0`),
- voluntad manual,
- migración de workspaces con `moveworkspacetomonitor` (solo hacia monitores
  activos; así las ventanas nunca se pierden ni caen en monitores apagados).

Llamadores: `monitor-restore.sh` (login, silent), `monitor-toggle.sh`
(Super+Z/X con "silent" + su aviso) y `monitor-watch.sh` (servidor de eventos
`socket2`: `monitoradded`/`monitorremoved` → aplica en el momento).

## Estados y qué pasa

| Estado | Resultado |
|---|---|
| Ambos ON | LG 1-5 (frente) · Toshiba 6-10 (izquierda). Barra: cada pantalla pinta los suyos |
| Solo LG | ws6-10 migrados al LG (funcionan por teclado, no se pintan en barra). Aviso de escritorio |
| Solo Toshiba | ws1-5 migrados al TV, que pasa a `0x0` como principal |
| Al reconectar | el script migra de vuelta sus workspaces; ventanas intactas |

## Waybar

- Workspaces como **números** (`format: "{id}"`); el bloque `format-icons`
  antiguo quedó vacío.
- Chip `custom/monitor` por barra con la marca (LG / TOSHIBA).

## Verificación hecha (2026-10-27)

- [x] ambos ON: LG `0x0` + Toshiba `-1920x0`; ws1-2→HDMI, ws6→DP
- [x] `monitor-toggle.sh DP-1` apagando: ws migró al LG
- [x] encendiendo: ws6 volvió a DP
- [x] política idempotente: `{"mode":"ambos","actions":[],"moves":[]}` en segunda corrida
- [x] waybar recargado con números

## Rollback

Backups locales: `hyprland.conf.bak-monitores`, `monitor-toggle.sh.bak-monitores`,
`monitor-restore.sh.bak-monitores`, `config.jsonc.bak-monitores` (todo en
`~/.config/{hypr,waybar}/`).
