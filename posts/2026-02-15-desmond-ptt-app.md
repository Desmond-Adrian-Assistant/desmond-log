# Building a Push-to-Talk App for My AI Assistant

**Feb 15, 2026 • Adrian**

I wanted to talk to Desmond — my AI assistant — hands-free from my phone. Not type, not tap through menus. Just hold a button and talk, walkie-talkie style. This is the story of building that.

**[GitHub: desmond-ptt](https://github.com/Desmond-Adrian-Assistant/desmond-ptt)** | **[Download APK](https://github.com/Desmond-Adrian-Assistant/desmond-ptt/releases/tag/v3.4)**

## The Problem

Desmond lives on Telegram. I talk to him all day — market briefs, research, random questions. But typing on a phone while walking or driving is a pain. Telegram has voice messages, sure, but I wanted something faster: a floating button that's always there, over any app, one-touch to talk.

## The Journey

### Attempt 1: Home Screen Widget

Android widgets seemed like the obvious choice. Slap a mic button on the home screen, done. Except Android widgets are basically static views that refresh on a timer. You can't do real-time audio recording from a widget. The `RemoteViews` API is intentionally limited — no custom touch handling, no streaming. Dead end.

### Attempt 2: Floating Overlay Button

The real solution: a foreground service with `TYPE_APPLICATION_OVERLAY`. This gives you a view that floats over everything. The tricky part was the UX:

- **Hold to record, release to send** — sounds simple, but you need to distinguish between taps, holds, and drags
- **Drag to reposition** — the button needs to be movable without accidentally triggering a recording
- **Smart threshold** — 15px of movement = drag (cancel recording). 400ms hold = start recording

The solution: a `postDelayed` Runnable that starts recording after the hold delay. If the finger moves past the drag threshold before the delay fires, cancel it. If recording already started and then you drag, cancel the recording. Clean UX.

### Attempt 3: Actually Sending to Telegram

First I used the Bot API — send audio to a bot that forwards it to Desmond. But bot messages show up as "from bot," not from me. Desmond's system sees bot-forwarded audio differently than a direct user voice message.

The fix: **TDLib** (Telegram Database Library). It lets you act as a full Telegram user client. Voice messages sent via TDLib appear as if you sent them yourself from the Telegram app.

### The TDLib Compilation Saga

This was the hardest part of the whole project. TDLib is a C++ library. To use it on Android, you need to cross-compile it for ARM using the Android NDK. The official instructions exist but gloss over several pain points:

1. **CMake version conflicts** — TDLib needs a specific CMake range that may conflict with what the NDK ships
2. **OpenSSL cross-compilation** — TDLib depends on OpenSSL, which also needs to be cross-compiled for Android ARM64
3. **The output** — you get `libtdjni.so` (~21MB per ABI) and `TdApi.java` (4.5MB, 141K lines of generated code)
4. **Four ABIs** — arm64-v8a, armeabi-v7a, x86, x86_64. I ended up shipping only arm64-v8a since the target device (Galaxy Z Fold) is ARM64

The whole TDLib build process took longer than writing the actual app. But the result is worth it — proper userbot voice messages.

## How It Works

```
Phone → Hold Button → Record Audio (AAC/M4A)
     → Release → TDLib sends as user voice message
     → Telegram → Desmond receives & transcribes with Whisper
     → AI responds in chat
```

The floating button service runs as a foreground service with a persistent notification. Visual states: idle (blue), recording (red pulse), sending (orange spin), success (green check), error (red X). Haptic feedback on record start/stop.

There's also a fallback chain: if TDLib auth isn't complete, it falls back to the Bot API. And an optional webhook endpoint for server-side Whisper transcription.

## v3.4: Making It Shareable

The original version had my API keys and bot tokens hardcoded everywhere. To open-source it, I:

- Created `AppConfig` with `EncryptedSharedPreferences` for all credentials
- Added a first-launch setup wizard (API ID → target chat → webhook)
- Made hold delay, min recording duration, and target chat configurable
- Moved the webhook server config to environment variables
- Security scrubbed every file

## Try It

If you have your own AI assistant on Telegram (or just want a floating PTT button):

1. Get API credentials from [my.telegram.org](https://my.telegram.org)
2. Download the [APK from GitHub](https://github.com/Desmond-Adrian-Assistant/desmond-ptt/releases/tag/v3.4)
3. Run through the setup wizard
4. Hold to talk

The code is MIT licensed. PRs welcome.

---

*This is part of the Desmond project — building an AI assistant that's actually useful in daily life. More at [desmond-log](https://ayedreeean.github.io/desmond-log/).*
