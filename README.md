# Voice Commander v2 — Architecture & User Guide

We inspected your live session logs (`journalctl --user -u voice-commander`) and found the exact root causes of every issue you experienced, then upgraded [voice-commander](file:///home/navalblues/.local/bin/voice-commander) and [config.toml](file:///home/navalblues/.config/voice-commander/config.toml) to **v2**:

---

## 1. Root Causes Found & How v2 Fixes Them

| Problem You Experienced | Root Cause in v1 Logs | Fix Deployed in v2 |
| :--- | :--- | :--- |
| **1 & 5. High latency & keeps waiting after you finish speaking** | The Samson GoMic condenser mic had a room ambient noise floor of `RMS ~880–1100`, above v1's `750` threshold. So every time you spoke, the mic never detected silence and stayed open until the **25-second timeout** (`duration: 25443ms`) before sending audio to Superwhisper! | **Adaptive Noise-Floor VAD + 0.85s Auto-Stop**: Now tracks a rolling median of room noise, starts recording at `RMS >= 2200`, uses hysteresis to ignore background room hum, and stops within **0.85 seconds** after you pause speaking. You can also say **"... over"** at the end of any sentence. |
| **2. Saying `"Computer open Chromium"` typed into Antigravity/Herdr instead of executing** | Because the mic was stuck open for 25s, multiple retries got merged into a single string (`"Open. Can you please open the browser? Computer, open the browser."` or `"Exit Computer Workspace Two. Computer Open Terminal."`). Since `"Computer"` wasn't the very first word of the 25s clip, v1 treated the whole block as dictation! | **Multi-Command Splitter & Conversational Extractor**: v2 splits compound utterances on wake words (`Computer`, `Assistant`, `Jarvis`, `Omarchy`) and sentence boundaries, strips polite wrappers (*"Can you please..."*), deduplicates retries, and **never** pastes a `"Computer ..."` command into an input box. |
| **3. No runtime feedback while speaking** | Superwhisper Cloud (`S1-Voice`) uploads the audio clip only when recording stops, so there was no visual indicator of live capture or when the mic would release. | **Live Runtime OSD Meter**: While you speak, an on-screen notification displays a live audio level bar (`Mic: [██████░░░░]`) and countdown (`Pause 0.8s to send`), switching to `"Processing Speech..."` the instant you pause. |
| **4. Personal Assistant for Browser Links, Navigation & App Launcher** | v1 only opened raw domains or ran basic Google searches, and `"Enter"` / `"see apps"` weren't mapped as direct commands. | **Agentic Browser & Desktop Assistant**: Added link-clicking by name (`"Computer, click on link <text>"`), page searching (`"find on page <text>"`), natural navigation (`"take me to <site/topic>"`), YouTube/GitHub search, and direct single-word `"Enter"` and `"See apps"` commands. |

---

## 2. New Voice Commands in v2

### Personal Browser Assistant (Navigating & Clicking Links by Voice)
| What You Say | What Happens |
| :--- | :--- |
| *"Computer, take me to Hacker News"* (or *YouTube, GitHub, Gmail, ChatGPT, Reddit, Wikipedia*) | Opens the site directly in Chromium |
| *"Computer, take me to <any topic>"* | Opens Chromium and searches Google for `<topic>` |
| *"Computer, click on link <text>"* (or *"open link <text>"*) | Uses Chromium Quick-Find (`Ctrl+F` -> `<text>` -> `Enter` -> `Escape` -> `Enter`) to focus and **click any link or button on the page** matching `<text>`! |
| *"Computer, find on page <text>"* | Highlights `<text>` on the current webpage |
| *"Computer, next match"* / *"previous match"* | Cycles through matches on the page (`Ctrl+G` / `Ctrl+Shift+G`) |
| *"Computer, search YouTube for <topic>"* | Searches YouTube directly |
| *"Computer, search GitHub for <topic>"* | Searches GitHub directly |
| *"Computer, tab two"* / *"switch to tab 3"* | Switches directly to browser tab 1–9 |
| *"Computer, go back"* / *"go forward"* | Browser history Back (`Alt+Left`) / Forward (`Alt+Right`) |

### Fast Single-Word & Inline Dictation Controls
| What You Say | What Happens |
| :--- | :--- |
| *"Enter"* / *"Press enter"* / *"Submit"* | Presses `Enter` immediately (no longer types the word `"Enter."`) |
| *"<your sentence> press enter"* | Types `<your sentence>` into Antigravity/Herdr/Browser and automatically presses `Enter`! |
| *"See apps"* / *"Show apps"* | Opens the Omarchy application launcher (`Super`) |
| *"Scratch that"* / *"Delete word"* | Deletes the last word (`Ctrl+Backspace`) |
| *"Clear input"* | Clears the current input line (`Ctrl+U`) |
| *"Don't listen right now"* / *"Stop listening"* | Immediately pauses Hands-Free VAD (useful if you get a phone call; press `Ctrl+Alt+Space` or run `voice-commander hands-free on` to resume) |
