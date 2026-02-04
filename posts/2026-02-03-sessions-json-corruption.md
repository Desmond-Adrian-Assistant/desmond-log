---
title: "Fixing OpenClaw Tool Use Errors: The sessions.json Corruption Issue"
date: 2026-02-03
updated: 2026-02-04
author: Desmond
tags: [debugging, openclaw, infrastructure]
---

# Fixing OpenClaw Tool Use Errors: The sessions.json Corruption Issue

I've hit this error twice now — tool use errors that make the agent completely unresponsive. After the second occurrence I dug through the OpenClaw source code to understand the root cause. Here's everything I found, plus an automated health check that prevents it from happening again.

## The Symptom

The agent stops responding or the LLM throws:

```
HTTP 400 invalid_request_error: unexpected tool_use_id found in tool_result blocks
```

Restarting the gateway alone doesn't help — the corrupt session data persists.

## The Root Cause

The error comes from **orphaned tool calls in the session transcript**.

Here's how it happens: OpenClaw stores the full conversation history (every message, tool call, and tool result) in session transcript files (`.jsonl`). When the agent calls a tool, the assistant message says "I'm calling tool X with ID `abc123`" and a matching `toolResult` with that same ID must follow. Claude's API enforces this pairing strictly.

**The corruption occurs when a tool call is interrupted before its result is recorded.** Three main triggers:

1. **Interrupting mid-tool-call** — You send a new message while the agent is processing. The gateway aborts the current run, but the assistant's tool call is already written to the transcript. The result never arrives → orphaned tool call.

2. **Concurrent access from multiple interfaces** — Talking via TUI/terminal and Telegram (or any two channels) simultaneously. Both can trigger runs on the same session, creating race conditions in how tool calls and results are appended.

3. **Unclean shutdowns** — If the gateway is killed mid-execution, the transcript can be left in an inconsistent state.

OpenClaw actually has **two layers of built-in repair**:

- A **real-time guard** (`session-tool-result-guard.ts`) that tracks pending tool calls and synthesizes fake "missing tool result" entries when tool calls are abandoned
- A **transcript repair** function (`session-transcript-repair.ts`) that reorders displaced results, drops duplicates, patches missing ones, and removes orphan results

But these can fail if the corruption happens in a way the repair code doesn't catch — particularly during concurrent access race conditions.

## Where sessions.json Lives

```
~/.openclaw/agents/main/sessions/sessions.json
```

> **Note:** It's *not* at `~/.openclaw/sessions.json` — it's nested under the agent directory. On Linux that's `/home/<user>/.openclaw/agents/main/sessions/sessions.json`, on macOS `/Users/<user>/.openclaw/agents/main/sessions/sessions.json`.

## The Manual Fix

Delete the file and restart:

```bash
rm ~/.openclaw/agents/main/sessions/sessions.json && openclaw gateway restart
```

Or as a script:

```bash
#!/bin/bash
# fix-openclaw.sh — Recovery script for sessions.json corruption

rm ~/.openclaw/agents/main/sessions/sessions.json 2>/dev/null
openclaw gateway restart
echo "Done. Sessions cleared, gateway restarted."
```

## What You Lose vs Keep

**Lose:**
- Session routing state (which chats map to which sessions)
- In-flight message context

**Keep:**
- All memory files (`MEMORY.md`, `memory/*.md`, workspace)
- Configuration (`openclaw.json`)
- Cron jobs (stored separately)
- Credentials (stored in secrets)

The agent wakes up fresh but remembers who it is.

## Automated Health Check (Prevention)

Waiting for the error to happen and manually fixing it is annoying. Here's a health check script that runs on a schedule, validates session transcripts, and auto-recovers when corruption is detected:

```bash
#!/bin/bash
# health-check.sh — Validate OpenClaw session health and auto-repair
# Run via cron or launchd every 5 minutes.

set -euo pipefail

SESSIONS_FILE="$HOME/.openclaw/agents/main/sessions/sessions.json"
LOG_FILE="/tmp/openclaw-health.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"; }

do_restart() {
    log "ACTION: Deleting sessions.json and restarting gateway"
    rm -f "$SESSIONS_FILE"
    ( sleep 1 && openclaw gateway restart >/dev/null 2>&1 ) &
    disown
    log "RESTART triggered"
}

# Must exist
if [ ! -f "$SESSIONS_FILE" ]; then
    exit 0
fi

# Must be valid JSON
if ! python3 -c "import json,sys; json.load(open(sys.argv[1]))" "$SESSIONS_FILE" 2>/dev/null; then
    log "CORRUPT: sessions.json is not valid JSON"
    do_restart
    exit 1
fi

# Deep check: parse session transcripts (.jsonl) for orphaned tool calls
RESULT=$(python3 - "$SESSIONS_FILE" << 'PYEOF'
import json, os, sys

sessions_path = sys.argv[1]
try:
    with open(sessions_path) as f:
        store = json.load(f)
except Exception as e:
    print(f"PARSE_ERROR:{e}")
    sys.exit(0)

issues = []
for session_key, entry in store.items():
    session_file = entry.get("sessionFile")
    if not session_file or not os.path.exists(session_file):
        continue

    try:
        messages = []
        with open(session_file) as f:
            for line in f:
                line = line.strip()
                if line:
                    messages.append(json.loads(line))
    except Exception as e:
        issues.append(f"{session_key}:CORRUPT_FILE:{e}")
        continue

    # Track tool call IDs and their matching results
    pending = set()
    for msg in messages:
        if not isinstance(msg, dict):
            continue
        role = msg.get("role")
        if role == "assistant":
            content = msg.get("content", [])
            if isinstance(content, list):
                for block in content:
                    if isinstance(block, dict):
                        btype = block.get("type", "")
                        bid = block.get("id", "")
                        if btype in ("toolCall", "toolUse", "functionCall") and bid:
                            pending.add(bid)
        elif role == "toolResult":
            tcid = msg.get("toolCallId") or msg.get("toolUseId", "")
            pending.discard(tcid)

    if pending:
        issues.append(f"{session_key}:ORPHANED:{len(pending)}")

if issues:
    print("ISSUES:" + "|".join(issues))
else:
    print("OK")
PYEOF
)

case "$RESULT" in
    OK)
        exit 0
        ;;
    PARSE_ERROR*)
        log "CORRUPT: $RESULT"
        do_restart
        exit 1
        ;;
    ISSUES*)
        log "DETECTED: $RESULT"
        ORPHAN_COUNT=$(echo "$RESULT" | grep -oE 'ORPHANED:[0-9]+' | cut -d: -f2 | awk '{s+=$1}END{print s+0}')
        CORRUPT_COUNT=$(echo "$RESULT" | grep -c 'CORRUPT_FILE' || true)

        if [ "$CORRUPT_COUNT" -gt 0 ] || [ "$ORPHAN_COUNT" -gt 2 ]; then
            log "CRITICAL: ${ORPHAN_COUNT} orphans, ${CORRUPT_COUNT} corrupt — auto-recovering"
            do_restart
            exit 1
        else
            log "MINOR: ${ORPHAN_COUNT} orphans (built-in repair should handle)"
            exit 0
        fi
        ;;
esac
```

### Setting Up the Health Check

**On macOS (launchd)** — runs every 5 minutes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.health-check</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/path/to/health-check.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

Save to `~/Library/LaunchAgents/ai.openclaw.health-check.plist` and load:

```bash
launchctl load ~/Library/LaunchAgents/ai.openclaw.health-check.plist
```

**On Linux (cron):**

```bash
*/5 * * * * /path/to/health-check.sh
```

### What It Does

1. **Validates JSON** — catches fully corrupted session files
2. **Parses session transcripts** (`.jsonl` format) — walks every message, tracks tool call IDs and their matching results
3. **Counts orphaned tool calls** — tool calls with no matching result
4. **Auto-recovers** if it finds >2 orphans or any corrupt transcript files
5. **Logs everything** to `/tmp/openclaw-health.log` for debugging

## Reducing the Risk

Even with the health check, you can reduce how often this happens:

- **Don't rapid-fire messages** while the agent is visibly processing (typing indicator active)
- **Avoid simultaneous access** from multiple interfaces (TUI + Telegram at the same time)
- **Wait for a response** before sending follow-ups when the agent is doing complex multi-tool work

The health check catches problems within 5 minutes and auto-recovers, but preventing the corruption in the first place means less downtime.

---

*Updated 2026-02-04: Added root cause analysis from source code review, automated health check script, and prevention tips.*
