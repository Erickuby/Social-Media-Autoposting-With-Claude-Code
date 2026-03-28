---
name: continue
description: "Reboot Memory — read latest session, check current state, verify environment, report status and suggest next action. Use when starting a session or saying 'continue', 'resume', 'where were we', 'what's next', 'pick up where we left off'."
---

# /continue — Reboot Memory

Execute these steps in order. Do not skip any.

---

## Step 1: Read Latest Session

Find and read the most recent file in the `sessions/` folder:

```bash
# Get the most recently modified session file
ls -t sessions/*.md 2>/dev/null | head -1
```

Read that file in full. Internalise:
- What was completed last session
- What was left in progress
- Any issues that were flagged
- What the suggested next action was

---

## Step 2: Read Current State

Read `current_state.json` from the project root.

This contains:
- `last_session` — path to the last session log
- `pending_tasks` — tasks carried over from last session
- `system_status` — last known environment status

---

## Step 3: Verify Environment

Run these checks silently — do not print raw output, just note pass/fail for the status report:

```bash
# Node.js (need 18+)
node --version

# npm packages installed
test -d node_modules && echo "INSTALLED" || echo "MISSING"

# Python (need 3.10+)
python --version 2>/dev/null || py --version 2>/dev/null

# Clip extractor dependencies
python -c "import mediapipe, cv2, numpy, filterpy, rapidfuzz; print('OK')" 2>/dev/null || echo "MISSING"

# FFmpeg
ffmpeg -version 2>/dev/null | head -1 || echo "NOT FOUND"

# API keys (check presence only — never display values)
grep -q "ZERNIO_API_KEY_ERIC=." .env 2>/dev/null && echo "SET" || echo "MISSING"
grep -q "ZERNIO_API_KEY_AIVISION=." .env 2>/dev/null && echo "SET" || echo "MISSING"
grep -q "KIE_API_KEY=." .env 2>/dev/null && echo "SET" || echo "MISSING"
grep -q "ELEVENLABS_API_KEY=." .env 2>/dev/null && echo "SET" || echo "MISSING"
```

---

## Step 4: Load Context

Internalise the goals and state from Steps 1 and 2. You are now aware of:
- What was done last session
- What is still pending
- Whether the environment is ready

---

## Step 5: Status Report

Reply to the user with this exact format:

```
## Session Resumed — [Current Date]

### System Readiness
- [✅/❌] Node.js [version]
- [✅/❌] npm packages
- [✅/❌] Python [version]
- [✅/❌] Clip Extractor
- [✅/❌] FFmpeg
- [✅/❌] Zernio (Eric Nwankwo — YouTube + Facebook)
- [✅/❌] Zernio (AI Vision Consulting — Instagram + TikTok)
- [✅/❌] KIE.ai
- [✅/❌] ElevenLabs

### Where We Left Off
[2-4 sentence summary of what was done last session and what was pending]

### Pending Tasks
1. [Task 1 from current_state.json]
2. [Task 2 from current_state.json]
(or "No pending tasks — ready for new work")

---

Would you like to proceed with [top pending task]?
```

---

## Rules

- Never display actual API key values
- If `sessions/` is empty or does not exist, say so and ask what to work on
- If `current_state.json` is missing or empty, note it and proceed with what you can infer from the session log
- Flag any missing dependencies immediately — do not proceed as if they are fine
