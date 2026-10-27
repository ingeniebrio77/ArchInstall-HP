# Layout master en Hyprland + bug kitty 0.49.2 (fullscreen fantasma)

> Fecha: 2026-10-27 · Hyprland 0.56.2 · kitty → 0.48.2 (downgrade)

## 1. Layout master 50/50

Problema previo: con `dwindle` las ventanas nuevas caían donde el ratón pudiera
(debajo/derecha, proporciones inconsistentes al combinar ventanas) y se "apilaban"
visualmente. Se cambia a **master global**.

Cambios en `~/.config/hypr/hyprland.conf` (backup previo en
`hyprland.conf.bak-master`):

```ini
general {
    layout = master             # ← añadido
    ... (gaps/borders sin cambio)
}

master {
    mfact = 0.5                 # maestra = 50% del ancho
    orientation = left          # maestra a la izquierda
    new_on_top = false          # nuevas al final de la pila
    new_status = slave          # las nuevas NO se vuelven maestra
}
```

Binds corregidos en el mismo archivo:

| Atajo | Antes | Ahora |
|---|---|---|
| `Super+T` | `layoutmsg, togglesplit` (no-op en master) | `layoutmsg, swapwithmaster` |
| `Super+K` | `kitty -e khal` (khal visto, no interactivo) | `kitty -e ikhal` |

> `Super+P` (pseudo) es de dwindle: con master queda como no-op inofensivo.

### Comportamiento verificado (hyprctl clients -j, ws vacío)

1. 1ª ventana → área completa (sin flag fullscreen).
2. 2ª → izquierda 50% + derecha 50% (mfact 0.5 exacto: 941/941 px en DP-1 1920).
3. 3ª → izquierda + derecha-arriba + derecha-abajo, **0 solapes**.

Nota: las ventanas **existentes** conservan su geometría vieja hasta que ese
workspace se re-tilea (abrir/cerrar una ventana ahí basta).

## 2. Bug kitty 0.49.2: primera ventana sale en fullscreen

### Síntoma
La primera kitty abierta en un workspace salía con `fullscreen=1` (no animación:
el flag persistía a los 6 s) y tapaba las ventanas tileadas debajo → efecto
"se solapan". Reproducible con `hyprctl dispatch exec kitty` pura.

### Diagnóstico (sondas con `[workspace N silent]`,ογ `hyprctl clients -j`)

| Prueba | fullscreen |
|---|---|
| kitty 0.49.2 + fish (config propio) | 1 |
| kitty 0.49.2 + `fish --no-config` | 1 |
| kitty 0.49.2 + `sleep` (sin shell) | 1 |
| kitty con `KITTY_SHELL_INTEGRATION=` vacío / `shell_integration=disabled` | 1 |
| kitty vía XWayland (`WAYLAND_DISPLAY=` → X11) | 1 |
| pavucontrol / zathura (GTK Wayland) | 0 |
| layout dwindle vs master | indiferente |

_NO_ era windowrules, binds, reglas de workspace (`hyprctl workspacerules`),
kitty.conf (`/etc/xdg/kitty` no existe, el propio es cosmético) ni scripts de
fish/hypr. kitty 0.49 (overhaul de Wayland "a la par de X11") **pide**
fullscreen al compositor al abrirse. kitty se actualizó 0.48.2→0.49.2 el
2026-10-20, mismo periodo del síntoma.

### Fix aplicado (2026-10-27)

Downgrade desde la caché de pacman (los 3 paquetes):

```bash
sudo pacman -U /var/cache/pacman/pkg/kitty-0.48.2-1-x86_64.pkg.tar.zst \
  /var/cache/pacman/pkg/kitty-terminfo-0.48.2-1-x86_64.pkg.tar.zst \
  /var/cache/pacman/pkg/kitty-shell-integration-0.48.2-1-x86_64.pkg.tar.zst
```

- snap-pac hizo snapshots pre/post automáticamente (snapper #306/#307).
- El kitty corriendo no se reinicia (la imagen queda en memoria); solo las
  ventanas nuevas usan 0.48.2.

Pinneo en `/etc/pacman.conf` hasta que salga fix upstream:

```ini
IgnorePkg = kitty kitty-terminfo kitty-shell-integration
```

Cuando se publique un kitty corregido: borrar esa línea, `sudo pacman -Syu kitty`
y re-verificar con: `hyprctl dispatch exec '[workspace 9 silent] kitty'` +
`hyprctl clients -j` → `fullscreen` debe ser `0`.
