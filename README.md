# SuperDictate 🎙️🤖

SuperDictate is a Linux-native, Wayland-compatible smart dictation tool. It uses a hybrid pipeline to achieve near-perfect transcription, auto-typing, and intelligent command routing.

1. **Local Transcription:** Captures audio using `arecord` and transcribes it locally with `faster-whisper` (turbo model).
2. **Intent Analysis & Cloud Refinement:** Sends the raw transcription to **Gemini 3 Flash**. The AI routes the audio to either:
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
- **RAG Search:** *"Busca en mis notas sobre TheIA"* (Uses the `vault_qa` RAG pipeline and shows a Zenity popup with the AI's answer).
- **Todoist Tasks:** *"Recuérdame llamar a cliente mañana"*
- **OpenClaw (Sherlock):** *"Dile a Sherlock que revise mi correo"*

## Usage
Trigger the script via a custom keyboard shortcut (e.g., `Alt+Z`). 
- First press: Starts recording (🔴)
- Second press: Stops, transcribes, routes, and executes/pastes the text (⏳ -> ✅).