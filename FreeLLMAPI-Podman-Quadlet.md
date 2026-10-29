# FreeLLMAPI en Podman (Quadlet) — Router unificado de APIs LLM gratuitas

> Instalado el 2026-10-29. Repo oficial: <https://github.com/tashfeenahmed/freellmapi> (licencia **MIT**).
> Ojo con los forks: el canónico es `tashfeenahmed/freellmapi`.

## ¿Qué es y qué cuesta?

Router auto-hospedado que unifica **34 providers gratuitos de LLM (~635 endpoints de modelos)** detrás de **un solo endpoint compatible con OpenAI** (`http://127.0.0.1:3001/v1`), con routing inteligente, failover automático y keys cifradas en reposo.

**No hay abono ni suscripción.** Es MIT y self-hosted: gratis para siempre. El único coste posible es opcional: si decides añadir keys de providers *de pago* para tener más cuota. OpenCode tampoco exige suscripción; los costes siempre son de los providers de modelos, no de las herramientas.

### Beneficios para este sistema

- **Más flujo de trabajo gratis:** agrega las cuotas free de muchos providers; cuando uno se satura (rate-limit), FreeLLMAPI salta al siguiente automáticamente.
- **Una sola conexión en OpenCode:** un único proveedor `freellmapi` en `opencode.jsonc` expone todos los modelos gratuitos.
- **Privacidad de keys:** las API keys de los providers se guardan cifradas (AES con `ENCRYPTION_KEY`) en SQLite local.
- **Sin Docker:** corre como servicio de usuario systemd rootless vía Quadlet (filosofía Arch/KISS).

## Arquitectura de la instalación

| Pieza | Ubicación | Notas |
|---|---|---|
| Quadlet (unidad) | `~/.config/containers/systemd/freellmapi.container` | Genera `freellmapi.service` (usuario) |
| Secrets / config | `~/.config/freellmapi/freellmapi.env` | **chmod 600** — contiene `ENCRYPTION_KEY` y `PORT=3001` |
| Datos (SQLite) | volumen `freellmapi-data` → `~/.local/share/containers/storage/volumes/freellmapi-data/` | Backup = copiar ese directorio con el servicio parado |
| Dashboard + API | `http://127.0.0.1:3001` | **Solo localhost** (single-user; nunca exponer a LAN/Internet) |
| Imagen | `ghcr.io/tashfeenahmed/freellmapi:latest` | rootless, en el storage del usuario |
| Auto-update | `podman-auto-update.timer` (usuario) | Equivale a Watchtower, sin socket privilegiado |
| Arranque en boot | `loginctl enable-linger renato` (Linger=yes) | El servicio sube aunque no haya sesión abierta |

## Operación diaria

```fish
systemctl --user status freellmapi      # estado
systemctl --user restart freellmapi     # reiniciar
systemctl --user stop freellmapi        # parar
journalctl --user -u freellmapi -f      # logs en vivo
podman ps                               # contenedor corriendo
podman auto-update --dry-run            # ver si hay imagen nueva
```

Tras editar el `.container`: `systemctl --user daemon-reload && systemctl --user restart freellmapi`.

## Configuración inicial (una sola vez, vía navegador)

1. Abrir `http://127.0.0.1:3001` (dashboard React).
2. En **Settings/Providers**: añadir las keys de las APIs gratuitas deseadas (Gemini, Groq, Mistral, Cerebras, OpenRouter free, etc.).
3. Generar la **unified key** (`freellmapi-...`) — es la key que usarán los clientes.

## Integración con OpenCode — HECHO (2026-10-30)

Proveedor `freellmapi` añadido a `~/.config/opencode/opencode.jsonc` (la unified key real vive SOLO ahí, chmod 600 — nunca en este repo):

```jsonc
"freellmapi": {
  "name": "FreeLLMAPI (local, modelos gratis)",
  "package": "@opencode/ai/providers/openai-compatible",
  "settings": {
    "baseURL": "http://127.0.0.1:3001/v1",
    "apiKey": "freellmapi-PEGA_AQUI_LA_UNIFIED_KEY"
  }
}
```

Modelos curados expuestos en el selector: `auto`, `gemini-3.8-flash`, `glm-5.3`, `qwen3.8-max`, `claude-sonnet-4-5`, `nemotron-3-ultra-550b`, `gpt-oss-120b` (el catálogo completo son ~284; añadir más = copiar un bloque `models`). Verificado: `/v1/chat/completions` con `auto` → enrutó a `qwen3.8-27b` y respondió en español.

Providers gratuitos configurados: Groq, Cerebras, OpenRouter, NVIDIA (pendiente SMS). Gemini requiere proyecto nuevo sin billing (cuenta prepagada agotada, error 402). GitHub Models fue retirado por GitHub el 2026-07-30; Mistral ya no tiene free tier de API (2026).

## Seguridad

- Endpoint atado a `127.0.0.1` — no abrir con `HOST_BIND=0.0.0.0` salvo LAN de confianza.
- `freellmapi.env` en 600: la `ENCRYPTION_KEY` cifra las keys de los providers. Si se pierde, las keys guardadas son irrecuperables (habría que re-introducirlas).
- Los snapshots de snapper en `/home` cubren el Quadlet, el `.env` y el volumen.

## Desinstalación

```fish
systemctl --user stop freellmapi
rm ~/.config/containers/systemd/freellmapi.container
systemctl --user daemon-reload
podman rmi ghcr.io/tashfeenahmed/freellmapi:latest
podman volume rm freellmapi-data        # solo si no se quiere conservar la config
rm ~/.config/freellmapi/freellmapi.env
```

## Verificación rápida (tras cualquier cambio)

```fish
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:3001/    # 200 = dashboard OK
curl -s http://127.0.0.1:3001/v1/models | head -c 100               # 401 sin key = API OK
systemctl --user is-active freellmapi                               # active
```

## Problemas conocidos

- **Keys con `fetch failed (UND_ERR_CONNECT_TIMEOUT)` en el dashboard** (2026-10-29): el namespace de red rootless del contenedor a veces arranca roto (sin salida a Internet, aunque el host sí tenga). Fix: `systemctl --user restart freellmapi` y pulsar "Comprobar todo" de nuevo. Si se repite a menudo, revisar la red rootless de Podman (pasta/netavark).
