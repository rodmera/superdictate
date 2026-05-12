# SuperDictate 🎙️✨

SuperDictate is a Linux-native, Wayland-compatible smart dictation tool. It uses a hybrid pipeline to achieve near-perfect transcription and auto-typing:

1. **Local Transcription:** Captures audio using `arecord` and transcribes it locally with `faster-whisper` (turbo model).
2. **Cloud Refinement:** Sends the raw transcription to **Gemini 3 Flash** for phonetic correction (understanding context, local names, and proper nouns).
3. **Auto-Typing:** Uses `evdev` to create a virtual kernel-level keyboard to paste the text directly into any application, bypassing Wayland's security restrictions on simulated keystrokes.

## Prerequisites
- `faster-whisper`
- `evdev` (Python package)
- `wl-clipboard`
- `/dev/uinput` permissions for the user

## Usage
Trigger the script via a custom keyboard shortcut (e.g., `Alt+Z`). 
- First press: Starts recording (🔴)
- Second press: Stops, transcribes, refines, and pastes the text (⏳ -> ✅).
EOF 2>&1