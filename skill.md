---
name: lumi-api
description: "Use this skill when the user wants to manage social media content via Lumi — uploading videos, publishing to TikTok/YouTube/Instagram, translating or dubbing content, checking account analytics, listing connected accounts, or managing localization tasks. All API calls use curl with Bearer token authentication. Trigger on: post to TikTok, upload video, translate video, dub video, check analytics, list my accounts, repurpose video, localize, list voices, publish to YouTube, schedule post."
---

# Lumi API Agent Skill

Lumi is a multi-platform social media management tool. This skill covers all public REST API operations via `curl`. Every request requires a Bearer token from your Lumi API key.

## Setup

```bash
export LUMI_API_KEY="lumi_YOUR_API_KEY_HERE"
export LUMI_BASE_URL="https://lumipath.cn"
```

All examples below use `$LUMI_API_KEY` and `$LUMI_BASE_URL`.

---

## 1. Connections — List Connected Social Accounts

```bash
# List all connected accounts
curl -s "$LUMI_BASE_URL/api/v1/connections" \
  -H "Authorization: Bearer $LUMI_API_KEY"

# Filter by platform (tiktok | youtube | instagram | facebook | x | reddit)
curl -s "$LUMI_BASE_URL/api/v1/connections?platform=tiktok" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```

**Response fields per connection:**
- `connectionId` — use this in social-posts and localization calls
- `platform`, `username`, `displayName`, `avatarUrl`, `status`, `createdAt`

---

## 2. Videos — Upload & List

### Register a video from a public URL
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/videos/upload" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/video.mp4",
    "filename": "my-video.mp4",
    "title": "My Video Title"
  }'
```
Returns: `{ "vid": "...", "url": "...", "title": "...", "status": "..." }`

### Download from Douyin / Bilibili / Xiaohongshu and upload
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/videos" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.douyin.com/video/XXXXXXX",
    "filename": "douyin-video.mp4",
    "title": "My Douyin Video"
  }'
```
Returns: `{ "url": "oss-hosted-url", "vid": "..." }` — use `url` in social-posts or localization.

### List videos in library
```bash
# Basic list (page 1, 10 per page)
curl -s "$LUMI_BASE_URL/api/v1/videos" \
  -H "Authorization: Bearer $LUMI_API_KEY"

# With pagination and search
curl -s "$LUMI_BASE_URL/api/v1/videos?page=1&limit=20&search=tutorial" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```
**Response:** `{ "videos": [...], "pagination": { "total", "page", "limit", "totalPages" } }`

**Video status values:** `uploaded | translating | translated | published | failed | scheduled`

---

## 3. Social Posts — Publish Video to Platforms

Supported platforms: `TIKTOK`, `YOUTUBE`, `INSTAGRAM`

### Publish to TikTok
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/social-posts" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platforms": ["TIKTOK"],
    "contentType": "VIDEO",
    "text": "Check out this video! #trending",
    "mediaUrls": ["https://your-video-url.mp4"],
    "tiktokConnectionIds": ["CONNECTION_ID_FROM_v1_connections_list"],
    "tiktokConfigs": [
      {
        "connectionId": "CONNECTION_ID",
        "allowComments": true,
        "allowDuet": false,
        "allowStitch": false,
        "privacyLevel": "PUBLIC_TO_EVERYONE"
      }
    ]
  }'
```

### Publish to YouTube
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/social-posts" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platforms": ["YOUTUBE"],
    "contentType": "VIDEO",
    "text": "Video description here",
    "mediaUrls": ["https://your-video-url.mp4"],
    "youtubeConnectionIds": ["CONNECTION_ID"],
    "youtubeConfigs": [
      {
        "connectionId": "CONNECTION_ID",
        "title": "My YouTube Video Title",
        "visibility": "public"
      }
    ]
  }'
```
**YouTube visibility options:** `public | private | unlisted`

### Publish to multiple platforms at once
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/social-posts" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platforms": ["TIKTOK", "YOUTUBE", "INSTAGRAM"],
    "contentType": "VIDEO",
    "text": "Cross-platform post caption",
    "mediaUrls": ["https://your-video-url.mp4"],
    "tiktokConnectionIds": ["TIKTOK_CONN_ID"],
    "youtubeConnectionIds": ["YOUTUBE_CONN_ID"],
    "instagramConnectionIds": ["INSTAGRAM_CONN_ID"],
    "youtubeConfigs": [{ "connectionId": "YOUTUBE_CONN_ID", "title": "Video Title", "visibility": "public" }]
  }'
```

**Response result status values:** `success | processing | failed`

**TikTok privacyLevel options:** `PUBLIC_TO_EVERYONE | MUTUAL_FOLLOW_FRIENDS | FOLLOWER_OF_CREATOR | SELF_ONLY`

---

## 4. Localization — Translate, Dub & Subtitle Videos

### List available TTS voices first
```bash
# All voices
curl -s "$LUMI_BASE_URL/api/v1/tts" \
  -H "Authorization: Bearer $LUMI_API_KEY"

# Filter by language and gender
curl -s "$LUMI_BASE_URL/api/v1/tts?language=en&gender=female" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```
Returns array of voices with `id` (use as `dubbingVoiceId`), `displayName`, `gender`, `language`, `locale`.

### Start a localization job
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/localization" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vid": "VIDEO_ID",
    "sourceUrl": "https://oss-video-url.mp4",
    "sourceLang": "zh",
    "targetLang": "en",
    "title": "My localization task",
    "dubbing": true,
    "dubbingVoiceId": 455,
    "backgroundMusic": 0,
    "subtitle": true,
    "subtitleStyle": "tpl-31-1-T",
    "ocr": false
  }'
```

**`backgroundMusic` options:** `0` = keep original | `1` = mute all | `2` = keep sound effects only

**With auto-publish on completion:**
```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/localization" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vid": "VIDEO_ID",
    "sourceUrl": "https://oss-video-url.mp4",
    "sourceLang": "zh",
    "targetLang": "en",
    "dubbing": true,
    "autoPublish": {
      "text": "Now available in English!",
      "tiktokConnectionIds": ["TIKTOK_CONN_ID"],
      "youtubeConnectionIds": ["YOUTUBE_CONN_ID"]
    }
  }'
```
Returns: `{ "data": { "taskId": "...", "vid": "...", "status": "..." } }`

### Check localization task status
```bash
# By taskId
curl -s "$LUMI_BASE_URL/api/v1/localization?taskId=TASK_ID" \
  -H "Authorization: Bearer $LUMI_API_KEY"

# List all tasks (with filters)
curl -s "$LUMI_BASE_URL/api/v1/localization?status=completed&page=1&limit=10" \
  -H "Authorization: Bearer $LUMI_API_KEY"

# Filter by video
curl -s "$LUMI_BASE_URL/api/v1/localization?vid=VIDEO_ID" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```

**Task status values:** `pending | running | completed | failed`

When `status=completed`, the response includes `outputUrl` — the translated video URL ready for publishing.

---

## 5. Repurpose — One-Shot: Crawl → Translate → Publish

The fastest workflow: provide a Chinese platform URL, translate/dub, and optionally auto-publish.

```bash
curl -s -X POST "$LUMI_BASE_URL/api/v1/repurpose" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sourceUrl": "https://www.douyin.com/video/XXXXXXX",
    "sourceLang": "zh",
    "targetLang": "en",
    "dubbing": true,
    "dubbingVoiceId": 455,
    "backgroundMusic": 2,
    "subtitle": true,
    "ocr": false,
    "autoPublish": {
      "text": "Amazing content! #viral",
      "tiktokConnectionIds": ["TIKTOK_CONN_ID"],
      "youtubeConnectionIds": ["YOUTUBE_CONN_ID"]
    }
  }'
```

**Supported source platforms:** `douyin.com`, `bilibili.com`, `xiaohongshu.com`, `xhslink.com`

Returns: `{ "data": { "taskId": "...", "vid": "...", "status": "..." } }` — poll with localization GET to track progress.

---

## 6. Insights — Account Analytics

```bash
# Get insights for a specific connected account
curl -s "$LUMI_BASE_URL/api/v1/insights?connectionId=CONNECTION_ID" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```

**Supported platforms:** TikTok, YouTube, Instagram, Facebook

Returns platform-specific metrics: followers, views, likes, comments, and more.

---

## Common Workflows

### Workflow A: Upload and post a video
```bash
# 1. Upload video
VID_RESP=$(curl -s -X POST "$LUMI_BASE_URL/api/v1/videos/upload" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/clip.mp4","title":"My clip"}')

VIDEO_URL=$(echo $VID_RESP | jq -r '.url')

# 2. Get connection ID
CONN_ID=$(curl -s "$LUMI_BASE_URL/api/v1/connections?platform=tiktok" \
  -H "Authorization: Bearer $LUMI_API_KEY" | jq -r '.connections[0].connectionId')

# 3. Post to TikTok
curl -s -X POST "$LUMI_BASE_URL/api/v1/social-posts" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"platforms\":[\"TIKTOK\"],\"contentType\":\"VIDEO\",\"text\":\"My post caption\",\"mediaUrls\":[\"$VIDEO_URL\"],\"tiktokConnectionIds\":[\"$CONN_ID\"]}"
```

### Workflow B: Repurpose a Douyin video to English TikTok
```bash
# One call does everything: crawl → translate → dub → publish
curl -s -X POST "$LUMI_BASE_URL/api/v1/repurpose" \
  -H "Authorization: Bearer $LUMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sourceUrl": "https://www.douyin.com/video/XXXXXXX",
    "sourceLang": "zh",
    "targetLang": "en",
    "dubbing": true,
    "subtitle": true,
    "autoPublish": {
      "text": "#trending #viral",
      "tiktokConnectionIds": ["YOUR_TIKTOK_CONN_ID"]
    }
  }'

# Poll until complete
curl -s "$LUMI_BASE_URL/api/v1/localization?taskId=TASK_ID" \
  -H "Authorization: Bearer $LUMI_API_KEY" | jq '.tasks[0].status'
```

### Workflow C: Check analytics for all accounts
```bash
# Get all connection IDs
curl -s "$LUMI_BASE_URL/api/v1/connections" \
  -H "Authorization: Bearer $LUMI_API_KEY" | jq '.connections[] | {connectionId, platform, displayName}'

# Then get insights per account
curl -s "$LUMI_BASE_URL/api/v1/insights?connectionId=CONN_ID" \
  -H "Authorization: Bearer $LUMI_API_KEY"
```

---

## Notes

- `connectionId` values come from `/api/v1/connections` — always fetch fresh before posting.
- After starting a localization/repurpose job, poll the task status every 10–30 seconds until `status=completed`.
- When `status=completed`, use `outputUrl` from the task as the `mediaUrls` for social posting.
- Default dubbing voice ID is `455` if you don't specify `dubbingVoiceId`.
- Subtitle style codes (e.g., `tpl-31-1-T`) can be browsed in the Lumi dashboard. Omit for no burnt-in subtitles.
- The `/api/v1/videos` POST endpoint (for Chinese platforms) auto-crawls to extract the real video URL; the upload result's `url` is the OSS-hosted copy ready for all downstream operations.
