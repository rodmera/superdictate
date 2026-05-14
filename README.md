# SuperDictate 🎙️🤖

SuperDictate is a Linux-native, Wayland-compatible smart dictation tool. It uses a hybrid pipeline to achieve near-perfect transcription, auto-typing, and intelligent command routing.

1. **Local Transcription:** Captures audio using `arecord` and transcribes it locally with `faster-whisper` (turbo model). A background pre-loader warms the model while you speak, eliminating cold-start latency (~3.7s saved).
2. **Intent Analysis & Cloud Refinement:** Sends the raw transcription to **Gemini Flash**. The AI routes the audio to either:
   - **Passive Dictation:** Fixes phonetic errors (understanding your local context, projects, and ecosystem names) and pastes the text.
   - **Active Command:** Parses the command and executes actions locally (e.g. open projects, query the Obsidian Vault, add Todoist tasks).
3. **Auto-Typing:** Uses `evdev` to create a virtual kernel-level keyboard to paste the text directly into any application, bypassing Wayland's security restrictions on simulated keystrokes.

## Prerequisites
- `faster-whisper`
- `evdev` (Python package)
- `wl-clipboard`
- `zenity` (for graphical popups)
- `/dev/uinput` permissions for the user

## Features
- **Smart Dictation:** Trigger and talk. It fixes bad Whisper transcriptions like "Watson" or "CreaEfecto".
- **Project Navigation:** *"Abre el proyecto CreaEfecto"*
- **Obsidian Vault:** *"Abre el vault"*
- **RAG Search:** *"Busca en mis notas sobre TheIA"* (Uses the `ask_sources` RAG pipeline across Obsidian vault + personal documents, shows a Zenity popup with the AI's answer).
- **Todoist Tasks:** *"Recuérdame llamar a cliente mañana"*
- **OpenClaw (Sherlock):** *"Dile a Sherlock que revise mi correo"*

## Architecture

```
First press  ──► arecord starts          ──► preloader.py (background)
                  audio captured              │
                                              ├─ loads WhisperModel (~3.7s)
                                              └─ waits for READY signal

Second press ──► arecord stops           ──► signals preloader via READY_FILE
                                              │
                                              ├─ transcribes audio (~5.4s, CPU)
                                              └─ writes result to RESULT_FILE
                  reads transcription ◄──────┘
                  calls Gemini Flash (intent analysis + refinement)
                  executes command or pastes text
```

Total latency from second press to paste: ~10s on CPU (vs ~14s without pre-loader).

## Usage
Trigger the script via a custom keyboard shortcut (e.g., `Alt+Z`). 
- First press: Starts recording (🔴) and pre-loads the Whisper model in background.
- Second press: Stops, transcribes, routes, and executes/pastes the text (⏳ -> ✅).