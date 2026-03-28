---
name: done
description: "Session Wrap-up — summarise progress, log system status, write session file, update current_state.json. Use when finished with a session or saying 'done', 'wrap up', 'close out', 'save everything'."
---

# /done — Session Wrap-up

Execute these steps in order. Do not skip any.

---

## Step 1: Summarise Progress

Review everything that happened in this conversation and produce a clear list of:
- Tasks completed in full
- Tasks started but not finished
- Any errors or blockers encountered

---

## Step 2: Track In-Progress Work

List any tasks that were started but not finished, with enough context that the next session can pick them up without re-explaining.

---

## Step 3: Log System Status

Run these checks silently and note the results:

```bash
# Node.js
node --version

# npm packages
test -d node_modules && echo "INSTALLED" || echo "MISSING"

# Python
python --version 2>/dev/null || py --version 2>/dev/null

# Clip extractor dependencies
python -c "import mediapipe, cv2, numpy, filterpy, rapidfuzz; print('ALL OK')" 2>/dev/null || echo "INCOMPLETE"

# FFmpeg
ffmpeg -version 2>/dev/null | head -1 || echo "MISSING"

# API keys (check presence only — never display values)
grep -q "ZERNIO_API_KEY_ERIC=." .env 2>/dev/null && echo "ZERNIO ERIC: SET" || echo "ZERNIO ERIC: MISSING"
grep -q "ZERNIO_API_KEY_AIVISION=." .env 2>/dev/null && echo "ZERNIO AIVISION: SET" || echo "ZERNIO AIVISION: MISSING"
grep -q "KIE_API_KEY=." .env 2>/dev/null && echo "KIE: SET" || echo "KIE: MISSING"
grep -q "ELEVENLABS_API_KEY=." .env 2>/dev/null && echo "ELEVENLABS: SET" || echo "ELEVENLABS: MISSING"
```

---

## Step 4: Write Session Log File

Save a new file at: `sessions/session-[YYYY-MM-DD]-[HHmm].md`

Use today's date and current time (24-hour format). Example: `sessions/session-2026-03-28-1430.md`

Use this exact template:

```markdown
# Session: [Short Title — 3-5 words describing what was done]

**Date:** YYYY-MM-DD
**Time:** HH:mm
**Type:** [Setup / Content / Feature / Bug Fix / Documentation]

## Completed
- [Task 1]
- [Task 2]

## In Progress
- [Task started but not finished — include enough context to resume]

## System Status
| Component | Status |
|-----------|--------|
| Node.js | [version] |
| npm packages | [Installed / MISSING] |
| Python | [version] |
| Clip Extractor | [OK / INCOMPLETE] |
| FFmpeg | [version] |
| Zernio (Eric) | [Set / MISSING] |
| Zernio (AI Vision) | [Set / MISSING] |
| KIE.ai | [Set / MISSING] |
| ElevenLabs | [Set / MISSING] |

## Content Produced
- [List any clips, thumbnails, carousels, posts — or "None this session"]

## Issues Found
- [Any problems encountered — or "None"]

## What's Next
- [Specific next actions for the following session]
```

---

## Step 5: Update current_state.json

Update the file `current_state.json` in the project root with this structure:

```json
{
  "last_session": "sessions/session-[YYYY-MM-DD]-[HHmm].md",
  "pending_tasks": [
    "[Task 1 carried over]",
    "[Task 2 carried over]"
  ],
  "system_status": "ready"
}
```

If there are no pending tasks, set `"pending_tasks": []`.

---

## Step 6: Final Response

Reply to the user with:

1. Confirmation that the session log has been written (include the filename)
2. Confirmation that `current_state.json` has been updated
3. A one-sentence preview of what to do next session

Format:

```
Session documented: sessions/session-[YYYY-MM-DD]-[HHmm].md
current_state.json updated with [N] pending task(s).

Next session: [One sentence describing the top priority task].
```

---

## Rules

- Never display actual API key values — check presence only
- Always write the session file before responding
- If nothing happened this session, still log it: "Exploratory session — no changes made"
- Do not push to GitHub automatically — ask the user first
