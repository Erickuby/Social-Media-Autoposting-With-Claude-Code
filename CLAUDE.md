# IX AI Agent Social Media Manager

The complete AI-powered content creation and distribution system. Create, edit, and publish content across 13+ platforms using Claude Code skills.

## Skills (17)

| Category | Skill | Triggers |
|----------|-------|----------|
| **Distribution** | `late-social-media` | "post to", "schedule post" |
| | `short-form-posting` | "post short", "post reel" |
| | `youtube-content-package` | "youtube package", "publish video" |
| **Visual Creation** | `thumbnail-creator` | "create thumbnail", "youtube thumbnail" |
| | `carousel-generator` | "create carousel", "carousel images" |
| | `document-carousel` | "document carousel", "LinkedIn PDF" |
| **Video Pipeline** | `clip-extractor` | "extract clips", "reframe video", "face tracking" |
| | `clip-selection` | "select clips", "find best clips" |
| | `edit` | "edit video", "edit clip" |
| | `video-editing` | (invoked by /edit -- router) |
| | `short-form-editing` | (invoked by router -- <90s) |
| | `long-form-editing` | (invoked by router -- 5+ min) |
| | `extracting-transcripts` | "transcribe", "extract transcript" |
| | `visual-overlay-creation` | "create illustration", "new visual" |
| **Voice** | `voice-dna` | "write post", "create caption", "write description", "draft post" |
| **Utility** | `video-upload-helper` | "compress video", "upload video" |
| | `content-analytics` | "check analytics" |

## Rules

1. **Never post without user approval.** Always show the content package and get explicit confirmation.
2. **Each platform gets unique content.** Same message, different wording. Never copy-paste across platforms.
3. **Ask for thumbnails** before posting video content to YouTube.
4. **Confirm titles** before posting to YouTube.
5. **Use the Late REST API** (curl) for posts that need platform-specific features.
6. **Use Late MCP tools** for simple single-platform posts.
7. **Voice DNA is mandatory.** Before writing ANY social media post, caption, description, script, or written content, load the `voice-dna` skill. All written output must match Eric's voice — direct, warm, practical, British English, never corporate or salesy. See `.claude/skills/voice-dna/SKILL.md` for the full profile.

## Session Commands

| Command | Purpose | Triggers |
|---------|---------|----------|
| `/continue` | Load context, check system, review state, suggest next action | "continue", "resume", "what's next", "where are we" |
| `/done` | Validate, sync docs, report, commit and push | "done", "wrap up", "commit this", "push it" |

## Pipeline Flow

```
/clip-selection > /clip-extractor > /transcribe > /edit > /post-short
```

1. **Clip Selection** -- Analyze transcript, score clips (5 categories, 0-100), select best moments
2. **Clip Extraction** -- Face-tracking reframe (16:9 to 9:16) via Python tool at `tools/clip_extractor/`
3. **Transcription** -- WhisperX GPU or AssemblyAI for word-level timestamps
4. **Editing** -- `/edit` routes to `video-editing` (router) then `short-form-editing` or `long-form-editing`
5. **Publishing** -- `/short-form-posting` or `/youtube-content-package` via Late/Zernio

## Clip Extractor — How It Works (IMPORTANT)

The clip extractor does **intelligent face-tracking reframe**, NOT a static center crop. Never use a raw ffmpeg `crop=` filter as a substitute.

**6-stage pipeline:**
1. **Face Detection** (MediaPipe BlazeFace) -- finds the face every N frames
2. **Signal Fusion** -- combines face + pose + saliency for robust tracking
3. **Temporal Smoothing** (Kalman/EMA) -- smooth, natural camera motion
4. **Deadzone Filtering** -- suppresses micro-jitter so the crop doesn't twitch
5. **Crop Calculation** -- centers the 9:16 window ON the face (mouth at center)
6. **Frame-by-frame rendering** -- interpolated crop positions piped through ffmpeg

**Dependencies:** `mediapipe`, `opencv-contrib-python`, `numpy`, `filterpy`, `pyyaml`, `rapidfuzz`

**All extracted clips go to:** `output/clips/YYYY-MM-DD-slug/` with a `clips-metadata.json`

## Editing Architecture (3-Skill System)

```
/edit > video-editing (ROUTER) > short-form-editing (<90s)
                                > long-form-editing (5+ min)
```

### Format Detection

| Duration | Format | Primary Component | Pop-Outs |
|----------|--------|------------------|----------|
| <90s (pipeline) | Short-form | ConceptOverlay | 8-15 |
| <90s (standalone) | Short-form | AppleStylePopup + FloatingCard | 5-7 |
| 5+ min | Long-form | ConceptOverlay | 30-40+ |

## Essential Commands

```bash
# Remotion
npm run studio                    # Preview in browser
npm run render -- <Id> out/x.mp4  # Render composition

# Clip Extractor
cd tools
python -m clip_extractor reframe --video input.mp4 --output clips/ --format 9x16
python -m clip_extractor batch --video source.mp4 --clips defs.json --output clips/
```

## Critical Editing Rules

1. **ALWAYS use WORDS data** (`.ts` files) for frame timing
2. **Pop-out at EXACT frame** the keyword is spoken
3. **ILLUSTRATION-FIRST** -- NO caption on most pop-outs
4. **illustrationSize:** 800 (no text), 700 (with text), 620 (CTA)
5. **NEVER use template illustrations** -- every pop-out needs UNIQUE visual metaphor
6. **ALWAYS use real images** for logos (`Img` + `staticFile()`)
7. **Long-form: NO zoom keyframes** (flat 1.0)
8. **Source video audio:** `volume={0}` ONLY when the source has burned-in captions or overlapping narration. For **podcast/interview clips where the speaker's voice IS the content**, use `volume={1}` — muting kills the entire clip.
9. **Background music** 0.02 volume, first 35s only
10. **SFX J-cut:** sound 2-3 frames BEFORE visual
11. **SFX variety** -- never repeat same sound consecutively

## Key APIs

### Late / Zernio (Social Posting + Media Storage)
- **Base URL:** `https://getlate.dev/api/v1`
- **Upload:** `POST /media/presign` then PUT to upload URL, use `publicUrl`
- **Post:** `POST /posts` with `platforms[]` array
- **MCP tools:** `accounts_list`, `posts_create`, `posts_list`, `media_presign`

### KIE.ai (AI Image Generation)
- **Base URL:** `https://api.kie.ai/api/v1`
- **Model:** `nano-banana-pro`
- **Create task:** `POST /jobs/createTask`
- **Poll status:** `GET /jobs/recordInfo?taskId={id}`

### ElevenLabs (Voiceover for Remotion videos)
- **Base URL:** `https://api.elevenlabs.io/v1`
- **API Key:** `$ELEVENLABS_API_KEY` (from .env)
- **Voice ID:** `$ELEVENLABS_VOICE_ID` (from .env) — Eric's cloned voice
- **Generate audio:** `POST /text-to-speech/{voice_id}`
- **Recommended model:** `eleven_multilingual_v2`
- **Output format:** `mp3_44100_128` — save to `output/audio/` before passing to Remotion
- **Usage:** Whenever a Remotion composition needs voiceover, generate the audio via ElevenLabs first, then reference the file in the composition using `staticFile()`

## Brand Assets

### AI Vision Consulting Logo
- **File:** `public/ai-vision-logo.png`
- **Usage:** `<Img src={staticFile('ai-vision-logo.png')} />`
- **RULE:** Every Remotion video must end with a 2-3 second logo CTA outro. Logo springs in, website URL fades in beneath it. See `.claude/skills/video-editing/SKILL.md` — "Logo CTA Outro" section for the full implementation.

### Platform Logos (`public/logos/`)
x.svg, instagram.svg, linkedin.svg, tiktok.svg, youtube.svg, facebook.svg, threads.svg, bluesky.svg, pinterest.svg, reddit.svg, snapchat.svg, telegram.svg, googlebusiness.svg

### SFX Library (`public/audio/`)
- Pop: `pop-402324.mp3`
- Whoosh variants: `whoosh-effect-382717.mp3`, `whoosh-bamboo-389752.mp3`, `whoosh-large-sub-384631.mp3`
- Background: `background-music.mp3` (volume 0.02)

## Remotion Components

| Component | Use Case | z-index |
|-----------|----------|---------|
| ConceptOverlay | Full-screen concept reveal | 100 |
| AppleStylePopup | Premium white popup | 100 |
| FloatingCard | Card overlay (speaker visible) | 20 |
| PlatformCascade | Logo grid reveal | 30 |
| KineticText | Animated text | 25 |

## File Conventions

- **Compositions:** `remotion/compositions/YourCompositionName.tsx`
- **Word data:** `remotion/data/your-composition-words.ts`
- **Illustrations:** `remotion/lib/illustrations/YourIllustrations.tsx`
- **Output:** `output/` organized by type (thumbnails/, carousels/, documents/, posts/)
- **FPS:** Always 30
- **Portrait:** 1080x1920
- **Landscape:** 1920x1080

## File Organization (All Content)

```
output/
  clips/YYYY-MM-DD-slug/          # Extracted/reframed clips + clips-metadata.json
  thumbnails/YYYY-MM-DD-slug/     # YouTube thumbnails
  carousels/YYYY-MM-DD-slug/      # AI image carousels
  documents/YYYY-MM-DD-slug/      # Document carousels (HTML > PDF > PNG)
  posts/YYYY-MM-DD-slug/          # Mixed-format posts
```

**CRITICAL:** All output goes in `output/` — never to Downloads, temp, or external locations.

## Tone

**All written content follows Eric's Voice DNA** (`.claude/skills/voice-dna/SKILL.md`).

Quick rules:
- Direct, warm, practical. Knowledgeable friend, not corporate trainer.
- British English always. No American spellings. No em dashes. No dollar signs.
- Open with bold claims or results, never "welcome back" or "in today's video."
- No banned words: leverage, synergy, game-changer, passionate, empower, deep dive.
- Guardrail included for any application or CV-related content.
- CTA is brief, at the end, not pushy.

## Session Tracking

Every `/done` writes a session log to `sessions/YYYY-MM-DD-slug.md`. This creates a running history of what was built, fixed, or posted each session.

```
sessions/
  2026-03-25-onboarding-setup.md
  2026-03-25-clip-extraction-fix.md
  ...
```

`/continue` reads the latest session log to pick up where the last session left off.

---

# Session Management System

## Command: /done
When I type `/done`, you must perform a "Session Wrap-up" by executing these steps:
1. **Summarize Progress:** Create a summary of all tasks completed in this session.
2. **Track In-Progress Work:** List any tasks that were started but not finished.
3. **Log System Status:** Note current environment readiness (e.g., are API keys for Zernio/KIE.ai working? Are dependencies installed?).
4. **Document to File:** Save this summary into a new file: `sessions/session-[YYYY-MM-DD]-[HHmm].md`.
5. **Update Carryover:** Update the `current_state.json` file in the root with the "Next Steps" so they are ready for the next session.
6. **Final Response:** Confirm the session is documented and provide a one-sentence preview of what we do next.

## Command: /continue
When I type `/continue`, you must "Reboot Memory" by executing these steps:
1. **Read Latest Session:** Locate the most recent file in the `sessions/` folder.
2. **Read Current State:** Read `current_state.json` to understand pending tasks.
3. **Verify Environment:** Silently check if `node`, `python`, and `ffmpeg` are still responsive.
4. **Load Context:** Internalize the goals from the previous session.
5. **Status Report:** Respond with:
   - A checkmark list of current system readiness.
   - A summary of where we left off.
   - A question asking if I'd like to proceed with the next pending task.
