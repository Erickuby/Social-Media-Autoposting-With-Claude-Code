# Social Media Autoposting With Claude Code

> AI-powered content creation and distribution system built on Claude Code.
> Create, edit, and publish content across YouTube, Instagram, TikTok, and Facebook — all from your coding agent.

---

## What This Is

This repo turns Claude Code into a fully capable social media manager. It connects your AI agent to your social media accounts via Zernio, gives it your brand voice, and equips it with 17 skills covering everything from video editing to posting.

Built and customised for **Eric | AI Vision Consulting** — AI specialist, career coach, and creator of the *Eric Explains AI* YouTube channel.

---

## What It Does

| Category | Capability |
|----------|------------|
| **Distribution** | Post to YouTube, Instagram, TikTok, Facebook via Zernio API |
| **Visual Creation** | Generate thumbnails (KIE.ai), carousels, and document PDFs |
| **Video Pipeline** | Face-tracking clip extraction, transcript-driven editing, Remotion rendering |
| **Voice & Brand** | All content written in Eric's voice — British English, direct, practical |
| **Session Memory** | `/continue` and `/done` commands for session state across conversations |

---

## Connected Accounts

| Profile | Platform | Handle |
|---------|----------|--------|
| Eric Nwankwo | YouTube | @ericexplainsai |
| Eric Nwankwo | Facebook | AI Vision Consulting page |
| AI Vision Consulting | Instagram | @aivisionconsulting |
| AI Vision Consulting | TikTok | @aivisionconsultingltd |

---

## Skills (17)

| Skill | What It Does | Trigger |
|-------|-------------|---------|
| `late-social-media` | Post to any connected platform via Zernio | "post to", "schedule post" |
| `short-form-posting` | YouTube Shorts, Reels, TikTok with unique captions per platform | "post short", "post reel" |
| `youtube-content-package` | Full YouTube package: title, description, tags, timestamps, thumbnail | "youtube package", "publish video" |
| `thumbnail-creator` | AI-generated YouTube thumbnails via KIE.ai | "create thumbnail" |
| `carousel-generator` | Multi-slide image carousels for LinkedIn and Instagram | "create carousel" |
| `document-carousel` | Educational documents as HTML to PDF to PNG slides | "document carousel", "LinkedIn PDF" |
| `clip-extractor` | Face-tracking reframe from 16:9 to 9:16 with MediaPipe + Kalman smoothing | "extract clips", "reframe video" |
| `clip-selection` | Score and select the best clips from a long-form transcript | "select clips", "find best clips" |
| `edit` | Entry-point router: detects format, dispatches to correct editing skill | "edit video", "edit clip" |
| `video-editing` | Shared component library and editing rules | (invoked by router) |
| `short-form-editing` | Polished short-form edits under 90 seconds with Remotion | (invoked by router) |
| `long-form-editing` | Long-form video editing (5+ minutes) with Remotion | (invoked by router) |
| `extracting-transcripts` | Word-level transcription for cutting and captions | "transcribe", "extract transcript" |
| `visual-overlay-creation` | Custom illustrations and motion graphics for video overlays | "create illustration", "new visual" |
| `voice-dna` | Eric's full brand voice guide — loaded before any written content | "write post", "create caption" |
| `video-upload-helper` | Compress and upload video files to Zernio storage | "compress video", "upload video" |
| `content-analytics` | Track post performance across platforms | "check analytics" |

---

## Requirements

- **Claude Code** — [claude.ai/code](https://claude.ai/code)
- **Node.js 18+** — for Remotion video engine
- **Python 3.10+** — for clip extractor (face detection and tracking)
- **FFmpeg** — for video processing (`winget install Gyan.FFmpeg` on Windows)
- **Zernio account** — [zernio.com](https://zernio.com) — free to start, connects your social accounts
- **KIE.ai account** — [kie.ai](https://kie.ai) — for AI image generation (optional, for thumbnails and carousels)

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/Erickuby/Social-Media-Autoposting-With-Claude-Code.git
cd Social-Media-Autoposting-With-Claude-Code
```

### 2. Install Node.js dependencies

```bash
npm install
```

This installs Remotion and all video composition packages.

### 3. Install Python dependencies

```bash
pip install -r tools/clip_extractor/requirements.txt
```

Packages installed:
- `mediapipe` — face detection (BlazeFace)
- `opencv-contrib-python` — video frame processing
- `numpy` — array operations
- `filterpy` — Kalman filter for temporal smoothing
- `pyyaml` — config parsing
- `rapidfuzz` — fuzzy text matching

Verify the install worked:
```bash
python -c "import cv2; print(cv2.__version__)"
```

If this prints a version number, you are ready. This is the most common failure point — if clip extraction produces a static crop instead of face tracking, re-run this check.

### 4. Create your `.env` file

Create a file named `.env` in the project root:

```env
# Eric Nwankwo — Facebook + YouTube
ZERNIO_API_KEY_ERIC=your-eric-zernio-api-key
ZERNIO_PROFILE_ID_ERIC=your-eric-profile-id

# AI Vision Consulting — Instagram + TikTok
ZERNIO_API_KEY_AIVISION=your-aivision-zernio-api-key
ZERNIO_PROFILE_ID_AIVISION=your-aivision-profile-id

# KIE.ai — AI image generation (thumbnails and carousels)
KIE_API_KEY=your-kie-api-key
```

Get your Zernio API key and Profile ID from your [Zernio dashboard](https://zernio.com).

### 5. Connect your social accounts in Zernio

Log into your Zernio dashboard and connect:
- YouTube (Eric Nwankwo profile)
- Facebook (Eric Nwankwo profile)
- Instagram (AI Vision Consulting profile)
- TikTok (AI Vision Consulting profile)

### 6. Open in Claude Code

```bash
claude
```

Claude Code reads all 17 skills automatically from the `.claude/` folder. No configuration needed.

---

## Session Commands

| Command | What It Does |
|---------|-------------|
| `/continue` | Reboot memory. Reads the latest session log, checks system readiness, and tells you exactly where you left off. |
| `/done` | Wrap up the session. Writes a log to `sessions/`, updates `current_state.json` with pending tasks. |

Use `/continue` at the start of every session and `/done` at the end. This gives Claude persistent memory across conversations.

---

## How to Use It

**Post to a platform:**
> "Post this to Instagram and TikTok: Just published Day 12 of my 30-Day AI Job Upgrade Challenge."

**Create a YouTube Shorts package:**
> "Create a full YouTube Shorts content package for this clip"

**Extract clips from a recording:**
> "Extract the best 3 clips from recording.mp4 and reframe for TikTok"

**Create a thumbnail:**
> "Create a YouTube thumbnail for my video about using Claude to write a Civil Service personal statement"

**Full YouTube package:**
> "Create a full YouTube package for my latest video — title, description, tags, timestamps, and thumbnail"

**Write a LinkedIn post:**
> "Write a LinkedIn post about today's AI job search tip"

---

## Video Pipeline

```
/clip-selection > /clip-extractor > /transcribe > /edit > /short-form-posting
```

1. **Clip Selection** — Analyse transcript, score clips across 5 categories, select best moments
2. **Clip Extraction** — Face-tracking reframe (16:9 to 9:16) via Python tool at `tools/clip_extractor/`
3. **Transcription** — WhisperX or AssemblyAI for word-level timestamps
4. **Editing** — `/edit` routes to `video-editing` then `short-form-editing` or `long-form-editing`
5. **Publishing** — `/short-form-posting` via Zernio API

---

## Clip Extractor — How It Works

The clip extractor does intelligent face-tracking reframe, not a static centre crop.

**6-stage pipeline:**
1. Face Detection (MediaPipe BlazeFace) — finds the face every N frames
2. Signal Fusion — combines face, pose, and saliency for robust tracking
3. Temporal Smoothing (Kalman/EMA) — smooth, natural camera motion
4. Deadzone Filtering — suppresses micro-jitter
5. Crop Calculation — centres the 9:16 window on the face
6. Frame-by-frame rendering — interpolated crop positions piped through FFmpeg

All extracted clips go to: `output/clips/YYYY-MM-DD-slug/` with a `clips-metadata.json`.

---

## File Structure

```
.
├── .claude/
│   ├── commands/
│   │   ├── continue.md         # /continue session command
│   │   └── done.md             # /done session command
│   └── skills/                 # 17 Claude Code skills
│       ├── voice-dna/          # Eric's brand voice and tone guide
│       ├── late-social-media/  # Zernio posting skill
│       ├── short-form-posting/ # Shorts/Reels/TikTok skill
│       └── ...
├── remotion/                   # Remotion video compositions
│   ├── compositions/           # Video composition files (.tsx)
│   ├── data/                   # Word timing data
│   └── playbook/               # Style guides and animation rules
├── tools/
│   └── clip_extractor/         # Python face-tracking pipeline
├── output/                     # All generated content (gitignored)
│   ├── clips/                  # Extracted and reframed clips
│   ├── thumbnails/             # YouTube thumbnails
│   ├── carousels/              # Image carousels
│   └── posts/                  # Mixed-format posts
├── sessions/                   # Session logs (one file per session)
├── examples/                   # Usage examples
├── current_state.json          # Short-term memory for /continue
├── CLAUDE.md                   # Main instruction file for Claude Code
└── .env                        # API keys (gitignored — never commit this)
```

---

## Brand Voice

All written content is produced in **Eric's Voice DNA** — loaded automatically from `.claude/skills/voice-dna/SKILL.md` before any content is written.

Quick rules:
- British English always. No American spellings. No em dashes. No dollar signs.
- Direct, warm, practical. Like a knowledgeable friend, not a corporate trainer.
- No "In today's fast-paced world." No "passionate professional." No "leverage."
- Guardrail included for any job application or Civil Service content.
- CTA is brief, at the end, not pushy.

---

## API Reference

### Zernio (Social Posting + Media Storage)
- Base URL: `https://getlate.dev/api/v1`
- Upload: `POST /media/presign` then `PUT` to upload URL
- Post: `POST /posts` with `platforms[]` array

### KIE.ai (AI Image Generation)
- Base URL: `https://api.kie.ai/api/v1`
- Model: `nano-banana-pro`
- Create task: `POST /jobs/createTask`
- Poll status: `GET /jobs/recordInfo?taskId={id}`

---

## Key Commands

```bash
# Preview Remotion video in browser
npm run studio

# Render a Remotion composition
npm run render -- <CompositionId> out/video.mp4

# Run clip extractor
cd tools
python -m clip_extractor reframe --video input.mp4 --output clips/ --format 9x16

# Batch extract clips
python -m clip_extractor batch --video source.mp4 --clips defs.json --output clips/
```

---

## About

Built for **Eric | AI Vision Consulting**
Newcastle upon Tyne, England, UK

- YouTube: [Eric Explains AI](https://youtube.com/@ericexplainsai)
- Instagram: [@aivisionconsulting](https://instagram.com/aivisionconsulting)
- TikTok: [@aivisionconsultingltd](https://tiktok.com/@aivisionconsultingltd)
- Website: [aivisionconsulting.co.uk](https://aivisionconsulting.co.uk)

Based on the IX AI Agent Social Media Manager framework by [Enrique Marq](https://youtube.com/@enriquemarq-0).
