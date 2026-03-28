---
name: short-form-posting
description: Posts short-form video (Shorts/Reels/TikTok) with platform-specific content packages. Transcribes video, creates tailored titles/descriptions/hashtags per platform, handles thumbnails, and publishes via Late API. Triggers on "post short", "post reel", "post shorts", "upload short", "short-form", "reels", "post to youtube instagram tiktok".
argument-hint: [video_file_path]
dependencies:
  - extracting-transcripts (for transcript extraction)
  - video-upload-helper (for compression + upload)
  - late-social-media (for account IDs, retry logic, API reference)
allowed-tools:
  - mcp__late__*
  - Bash(*)
  - AskUserQuestion
  - Read
---

# Short-Form Video Posting

Post short-form video content (YouTube Shorts, Instagram Reels, TikTok) with unique, transcript-driven content per platform.

---

## Workflow Overview

```
1. Transcribe video (WhisperX)
2. Confirm profile (Enrique or Trejon)
3. Create platform-specific content package
4. Show package to user for approval
5. Upload video to Late storage
6. Post via REST API (single multi-platform request)
7. Verify all posts succeeded
```

---

## Step 1: Transcribe the Video

Use WhisperX (preferred) via the extracting-transcripts skill:

```bash
py -3.11 "C:\Users\enriq\Downloads\transcribe_whisperx.py" "VIDEO_PATH"
```

Read the transcript output to understand:
- **Core message:** What problem does the video solve?
- **Key points:** What are the 2-4 main takeaways?
- **CTA mentioned:** What does the speaker tell viewers to do? (link in bio, free trial, etc.)
- **Hook:** What's the opening line/stat that grabs attention?

This transcript drives ALL content creation. Do not write generic descriptions.

---

## Step 2: Confirm Profile

Ask the user which profile/platform to post to. Use the correct API key per profile.

**Eric Nwankwo accounts** — API key: `ZERNIO_API_KEY_ERIC`
| Platform | Username | Account ID |
|----------|----------|------------|
| YouTube | @ericexplainsai | `69c75bf86cb7b8cf4ca792a3` |
| Facebook | AI Vision Consulting (page) | `69c75bec6cb7b8cf4ca79296` |

**AI Vision Consulting accounts** — API key: `ZERNIO_API_KEY_AIVISION`
| Platform | Username | Account ID |
|----------|----------|------------|
| Instagram | @aivisionconsulting | `69c75c896cb7b8cf4ca793bd` |
| TikTok | @aivisionconsultingltd | `69c75c7e6cb7b8cf4ca793a8` |

**Always verify with `mcp__late__accounts_list` before posting.**

---

## Step 3: Create Platform-Specific Content

**CRITICAL: Each platform MUST have unique wording.** Same core message, different phrasing. Never copy-paste the same text across platforms.

### YouTube Shorts

**Title (3 options for user to pick):**
- Max 70 characters
- Front-load the keyword/hook
- Include power words: Free, Proven, Easy, Fast
- Patterns that work:
  - "[Stat] — Here's Why" (curiosity)
  - "Stop [Bad Thing] — Do This Instead" (contrarian)
  - "How [Outcome] in [Timeframe]" (how-to)

**Description:**
```
[Hook sentence — restate the problem or stat from the video]

[2-3 sentences expanding on the value/solution. Reference specific points from the transcript.]

[CTA — use the exact CTA mentioned in the video, e.g., "Link in bio to try the IX system free"]

My Links:
Subscribe: https://www.youtube.com/@enriquemarq-0
LinkedIn: https://www.linkedin.com/in/enrique-marq-756191319/
Instagram: https://www.instagram.com/kikefuturo_/

#Hashtag1 #Hashtag2 #Hashtag3
```

**Tags:** Array of 10-15 researched keywords
- Primary (5-7): High-volume, directly relevant to the video topic
- Secondary (3-5): Related topics, medium volume
- Long-tail (2-3): Specific phrases people search for

**firstComment:** Engagement question directly related to the video topic
```
"What's the #1 thing killing your [topic] results? Drop it below."
```

**categoryId:** Match to content
- `"28"` Science & Technology
- `"22"` People & Blogs
- `"26"` Howto & Style
- `"27"` Education

---

### Instagram Reels

**Caption — KEEP IT MINIMAL:**
- **1-3 sentences maximum** (hook + value + CTA)
- **1 call to action** (link in bio / follow / save)
- **~3 hashtags only** (targeted, not generic)
- The caption should be readable in 2 seconds

**Format:**
```
[One punchy sentence about the problem/solution from the video.]
[Optional second sentence with the key insight.]
[CTA — "Link in bio" or "Save this for later."]

#hashtag1 #hashtag2 #hashtag3
```

**firstComment:** Engagement prompt
```
"Save this and share it with someone who needs to hear it."
```

**Thumbnail options:**
- `thumbOffset`: Milliseconds from video start (ask user which moment looks best)
- `instagramThumbnail`: Custom cover image URL (upload via presign if user provides one)

**IMPORTANT:** Instagram caption must be DIFFERENT wording from YouTube. Same message, different angle.

---

### TikTok

**Caption — SHORT AND PUNCHY:**
- **1-2 sentences max**
- **1 CTA** (link in bio / follow for more)
- **3-5 hashtags** (targeted)
- TikTok users scroll fast — every word must earn its place

**Format:**
```
[Punchy hook or stat.] [Key takeaway in one sentence.] Link in bio.

#hashtag1 #hashtag2 #hashtag3 #hashtag4 #hashtag5
```

**Required platformSpecificData:**
```json
{
  "privacy_level": "PUBLIC_TO_EVERYONE",
  "allow_comment": true,
  "allow_duet": true,
  "allow_stitch": true,
  "content_preview_confirmed": true,
  "express_consent_given": true,
  "video_cover_timestamp_ms": 5000
}
```

**IMPORTANT:** TikTok caption must be DIFFERENT wording from both YouTube and Instagram.

---

### Content Variation Rules

| Aspect | YouTube Shorts | Instagram Reels | TikTok |
|--------|---------------|-----------------|--------|
| **Tone** | Professional, detailed | Casual, direct | Punchy, conversational |
| **Description length** | 4-6 sentences + links | 1-3 sentences | 1-2 sentences |
| **Hashtags** | 3-5 in description + tags field | ~3 in caption | 3-5 in caption |
| **CTA style** | Subscribe / Comment / Link in bio | Save / Share / Link in bio | Follow / Link in bio |
| **First comment** | Engagement question | Save/share prompt | Not supported via API |

---

## Step 4: Show Content Package to User

Present all 3 platforms clearly. Use this format:

```
--- YOUTUBE SHORTS ---
Title options:
  1. [Title option 1]
  2. [Title option 2]
  3. [Title option 3]

Description:
[Full description]

Tags: [tag1, tag2, tag3, ...]
First comment: [comment text]

--- INSTAGRAM REELS ---
Caption:
[Full caption with hashtags]

First comment: [comment text]
Thumbnail: [thumbOffset suggestion or ask for custom]

--- TIKTOK ---
Caption:
[Full caption with hashtags]

Cover frame: [suggested timestamp in ms]
```

**Ask for:**
1. Title selection (YouTube — pick 1 of 3)
2. Thumbnail preference per platform
3. Any edits to captions
4. Explicit approval to post

**NEVER post without user saying "yes" or "publish" or "go ahead".**

---

## Step 5: Upload Video

### Check size
```powershell
(Get-Item "VIDEO_PATH").Length / 1MB
```

- Under 500MB: Upload directly
- Over 500MB: Use `video-upload-helper` skill to compress first

### Upload to Late storage
```bash
# Get presigned URL
curl -s -X POST "https://getlate.dev/api/v1/media/presign" \
  -H "Authorization: Bearer $LATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"filename": "video.mp4", "contentType": "video/mp4"}'

# Upload
curl -X PUT "$UPLOAD_URL" \
  -H "Content-Type: video/mp4" \
  --upload-file "VIDEO_PATH" \
  --progress-bar
```

### Upload thumbnail (if custom cover image provided)
```bash
curl -s -X POST "https://getlate.dev/api/v1/media/presign" \
  -H "Authorization: Bearer $LATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"filename": "thumbnail.jpg", "contentType": "image/jpeg"}'
```

---

## Step 6: Post via REST API

Use a single multi-platform request with `customContent` per platform:

```bash
curl -s -X POST "https://getlate.dev/api/v1/posts" \
  -H "Authorization: Bearer $LATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "[FALLBACK_CONTENT]",
    "mediaItems": [{
      "url": "[VIDEO_PUBLIC_URL]",
      "thumbnail": "[YT_THUMBNAIL_URL]",
      "containsSyntheticMedia": true,
      "instagramThumbnail": "[IG_COVER_URL]"
    }],
    "platforms": [
      {
        "platform": "youtube",
        "accountId": "[YT_ACCOUNT_ID]",
        "customContent": "[YOUTUBE_DESCRIPTION]",
        "platformSpecificData": {
          "title": "[SELECTED_TITLE]",
          "visibility": "public",
          "tags": ["tag1", "tag2", "..."],
          "firstComment": "[YT_FIRST_COMMENT]",
          "categoryId": "[CATEGORY_ID]"
        }
      },
      {
        "platform": "instagram",
        "accountId": "[IG_ACCOUNT_ID]",
        "customContent": "[IG_CAPTION]",
        "platformSpecificData": {
          "shareToFeed": true,
          "firstComment": "[IG_FIRST_COMMENT]",
          "thumbOffset": [THUMB_OFFSET_MS]
        }
      },
      {
        "platform": "tiktok",
        "accountId": "[TT_ACCOUNT_ID]",
        "customContent": "[TIKTOK_CAPTION]",
        "platformSpecificData": {
          "privacy_level": "PUBLIC_TO_EVERYONE",
          "allow_comment": true,
          "allow_duet": true,
          "allow_stitch": true,
          "content_preview_confirmed": true,
          "express_consent_given": true,
          "video_cover_timestamp_ms": [COVER_MS]
        }
      }
    ],
    "publishNow": true
  }'
```

**Reading the API key at runtime:**
```bash
# Extract from Claude Code config
LATE_API_KEY=$(python -c "import json; print(json.load(open('C:/Users/enriq/.claude.json'))['mcpServers']['late']['env']['LATE_API_KEY'])")
```

---

## Step 7: Verify Posts

After posting, verify each platform succeeded:

```
mcp__late__posts_list(status=published, limit=5)
```

**Check that you see entries for all 3 platforms.** If any are missing, follow the retry protocol from `late-social-media` skill:

1. Check `posts_list` — maybe it posted despite timeout
2. Only retry if confirmed NOT published
3. Maximum 3 attempts per platform
4. STOP and report after 3 failures

Report to user:
- Post IDs for each platform
- Platform URLs (from `platformPostUrl` in response)
- Any failures

---

## Example: Complete Package

*Based on a video about cold outreach with the hook "99% of your cold messages never get a response"*

### YouTube Shorts
```
Title: 99% of Your Cold Messages Get Ignored — Here's Why

Description:
Most cold outreach fails because of 3 missing pieces: infrastructure, personalization, and reply speed.

Companies fixing all three are booking 20-30 meetings per week with multi-channel campaigns that get 3x more replies.

Try the IX system free — link in bio.

My Links:
Subscribe: https://www.youtube.com/@enriquemarq-0
LinkedIn: https://www.linkedin.com/in/enrique-marq-756191319/
Instagram: https://www.instagram.com/kikefuturo_/

#ColdOutreach #B2BSales #AIAgents

Tags: ["cold outreach", "outreach strategy", "AI sales agents", "multi-channel outreach",
"B2B lead generation", "cold email tips", "sales automation", "booking meetings",
"outbound sales", "SDR tips", "AI outreach", "email deliverability"]

firstComment: What's the #1 thing killing your outreach results? Drop it below.
categoryId: "28"
```

### Instagram Reels
```
Caption:
Your cold messages are getting ignored because you're missing 3 things. Fix them and book 20-30 meetings a week. Link in bio.

#coldoutreach #b2bsales #aisales

firstComment: Save this and share it with someone who needs to hear it.
thumbOffset: 3000
```

### TikTok
```
Caption:
99% of cold messages fail. Here's the 3 things you're missing. Link in bio to try it free.

#coldoutreach #salestips #aiagents #b2b #outreach

video_cover_timestamp_ms: 3000
```

---

## Social Links (Eric Nwankwo / AI Vision Consulting)

```
YouTube: https://www.youtube.com/@ericexplainsai
Instagram: https://www.instagram.com/aivisionconsulting
TikTok: https://www.tiktok.com/@aivisionconsultingltd
Facebook: https://facebook.com/1039260889266180
```
