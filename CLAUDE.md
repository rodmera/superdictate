# SuperDictate — Notas para Claude Code

## Port a macOS (pendiente)

El core (Whisper, pre-loader, Gemini) es multiplataforma y no necesita cambios.
Solo hay que reemplazar 5 dependencias Linux-específicas:

| Componente | Linux | macOS |
|---|---|---|
| Grabación | `arecord` | `sounddevice` + `soundfile` (Python) o `sox rec` |
| Paste | `evdev` / `UInput` | `pyautogui` (Quartz, funciona directo) |
| Portapapeles | `wl-copy` | `pbcopy` |
| Notificaciones | `notify-send` | `osascript -e 'display notification...'` |
| Abrir apps/archivos | `xdg-open` | `open` |
| Diálogo RAG | `zenity` | `osascript` AppleScript dialog |

**Approach recomendado:** abstraer las 5 funciones en `platform_utils.py` con
implementaciones `linux` / `macos`. El script `super-dictate` principal no
necesita cambios estructurales.

**No reescribir en Swift.** Python en macOS tiene soporte nativo completo.
Swift solo tendría sentido para distribución en Mac App Store, lo que implicaría
sandboxing y haría el keystroke injection más difícil.

**Esfuerzo estimado:** 2-3 horas.

## Features implementadas

| Feature | Cómo activar |
|---|---|
| Modo raw (bypass Gemini) | `--mode raw` |
| Modo vault (guardar en Obsidian) | `--mode vault` — llama `capture_note()` del skill obsidian-capture |
| Append al portapapeles | `--append` — lee `wl-paste` y prepende al texto final |
| Historial | `--history [N]` — imprime en terminal las últimas N entradas del JSONL |
| Auto-stop por silencio | `silence_timeout: N` en config — usa sox + silence_watcher.py |
| Push-to-talk | `ptt.py` daemon — monitorea tecla configurable con evdev |

## Archivos del proyecto

| Archivo | Función |
|---|---|
| `super-dictate` | Script principal |
| `preloader.py` | Pre-carga Whisper en background |
| `silence_watcher.py` | Monitorea PID de sox y dispara stop cuando termina |
| `ptt.py` | Daemon push-to-talk |
| `indicator.py` | Icono de tray mientras graba |

## Configuración

Config en `~/.config/superdictate/config.json` (se crea automáticamente al primer uso):
- `language`: idioma para Whisper (ej. "es", "en", "pt"). Pasa al pre-loader vía `/tmp/super-dictate-lang`.
- `default_mode`: modo por defecto si no se pasa `--mode`.
- `vocabulary`: lista de términos propios inyectados al prompt de Gemini (nombres, siglas, marcas).
- `modes`: dict de modos. Cada modo tiene `name` y `prompt` que se añade al prompt de refined_text.

Para agregar un modo custom, editar el JSON y añadir una clave nueva bajo `modes`.
Los modos se activan con `--mode <clave>` o se configuran como `default_mode`.

## Arquitectura — Pre-loader

- `preloader.py` corre en background desde la primera pulsación
- Carga WhisperModel turbo (~3.7s) mientras el usuario habla
- Señaliza vía `/tmp/super-dictate-audio-ready` al parar la grabación
- Latencia post-segunda-pulsación: ~10s (vs ~14s sin pre-loader)
- Bottleneck restante: inferencia CPU ~5.4s (irreducible sin GPU/modelo más pequeño)

## Paste en Linux/Wayland

Se usa `evdev`/`UInput` (Ctrl+Shift+V) porque en GNOME Wayland no hay
alternativa universal:
- `wtype` requiere protocolo wlroots (no disponible en GNOME)
- `ydotool` versión repos Ubuntu no soporta Unicode sin daemon
- `gdbus Shell.Eval` deshabilitado en GNOME moderno
