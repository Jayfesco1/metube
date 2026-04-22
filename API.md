# MeTube API Documentation

MeTube provides a REST API and a Socket.IO interface for managing downloads and subscriptions.

## REST API

All endpoints are relative to the `URL_PREFIX` configured in your environment (default is `/`).

### Downloads

#### `POST /add`
Adds a new download to the queue.

**Request Body (JSON):**
- `url` (String, Required): The URL to download.
- `download_type` (String, Required): One of `video`, `audio`, `captions`, `thumbnail`.
- `quality` (String, Required):
    - For `video`: `best`, `2160`, `1440`, `1080`, `720`, `480`, `360`, `240`, `worst`.
    - For `audio`: `best`, `320`, `192`, `128`.
- `format` (String, Optional):
    - For `video`: `any`, `mp4`, `ios`.
    - For `audio`: `m4a`, `mp3`, `opus`, `wav`, `flac`.
    - For `captions`: `srt`, `txt`, `vtt`, `ttml`, `sbv`, `scc`, `dfxp`.
    - For `thumbnail`: `jpg`.
- `codec` (String, Optional): For video downloads. One of `auto`, `h264`, `h265`, `av1`, `vp9`.
- `folder` (String, Optional): Subfolder within the download directory.
- `custom_name_prefix` (String, Optional): Prefix for the filename.
- `playlist_item_limit` (Integer, Optional): Limit the number of items if the URL is a playlist.
- `auto_start` (Boolean, Optional): Default `true`. If `false`, the download is added to the `pending` list.
- `split_by_chapters` (Boolean, Optional): Whether to split the video by chapters.
- `chapter_template` (String, Optional): Output template for chapters.
- `subtitle_language` (String, Optional): Default `en`.
- `subtitle_mode` (String, Optional): One of `auto_only`, `manual_only`, `prefer_manual`, `prefer_auto`.
- `ytdl_options_presets` (Array of Strings, Optional): Names of configured presets.
- `ytdl_options_overrides` (Object, Optional): Custom yt-dlp options (if enabled by `ALLOW_YTDL_OPTIONS_OVERRIDES`).

**Response (JSON):**
- Success: `{"status": "ok"}`
- Error: `{"status": "error", "msg": "reason"}`

---

#### `GET /history`
Returns the download history.

**Response (JSON):**
An object containing three lists of Download objects:
```json
{
  "done": [],
  "queue": [],
  "pending": []
}
```

---

#### `POST /delete`
Removes downloads from history or queue.

**Request Body (JSON):**
- `ids` (Array of Strings, Required): List of download identifiers (usually URLs).
- `where` (String, Required): Either `queue` (for active/pending) or `done` (for completed/failed).

**Response (JSON):**
- `{"status": "ok"}`

---

#### `POST /start`
Starts pending downloads.

**Request Body (JSON):**
- `ids` (Array of Strings, Required): List of download identifiers from the `pending` list.

**Response (JSON):**
- `{"status": "ok"}`

---

#### `POST /cancel-add`
Cancels a large "add" operation (e.g., adding a huge playlist).

**Response (JSON):**
- `{"status": "ok"}`

---

### Subscriptions

#### `POST /subscribe`
Creates a new subscription.

**Request Body (JSON):**
Same as `POST /add`, plus:
- `check_interval_minutes` (Integer, Optional): How often to check for new videos (default 60).

**Response (JSON):**
- `{"status": "ok", "subscription": { ... }}`

---

#### `GET /subscriptions`
Lists all current subscriptions.

**Response (JSON):**
- Array of subscription objects.

---

#### `POST /subscriptions/update`
Updates an existing subscription.

**Request Body (JSON):**
- `id` (String, Required): The subscription ID.
- `enabled` (Boolean, Optional): Enable or disable the subscription.
- `check_interval_minutes` (Integer, Optional): Update the check interval.
- `name` (String, Optional): Update the subscription name.

**Response (JSON):**
- `{"status": "ok", "subscription": { ... }}`

---

#### `POST /subscriptions/delete`
Deletes subscriptions.

**Request Body (JSON):**
- `ids` (Array of Strings, Required): List of subscription IDs.

**Response (JSON):**
- `{"status": "ok"}`

---

#### `POST /subscriptions/check`
Forces an immediate check for new content in subscriptions.

**Request Body (JSON):**
- `ids` (Array of Strings, Optional): List of subscription IDs to check. If omitted, all enabled subscriptions are checked.

**Response (JSON):**
- `{"status": "ok"}`

---

### System & Configuration

#### `GET /presets`
Returns available yt-dlp option presets.

**Response (JSON):**
- `{"presets": ["preset_name1", "preset_name2"]}`

---

#### `GET /version`
Returns the MeTube and yt-dlp versions.

**Response (JSON):**
- `{"yt-dlp": "YYYY.MM.DD", "version": "..."}`

---

#### `POST /upload-cookies`
Uploads a cookies file.

**Request Body:** Multipart form with a `cookies` field.

**Response (JSON):**
- `{"status": "ok", "msg": "..."}`

---

#### `POST /delete-cookies`
Deletes the uploaded cookies file.

**Response (JSON):**
- `{"status": "ok"}`

---

#### `GET /cookie-status`
Returns whether cookies are currently present.

**Response (JSON):**
- `{"status": "ok", "has_cookies": boolean}`

---

## Socket.IO Events

MeTube uses Socket.IO for real-time updates.

### Server to Client (Emits)

- `all`: Emitted on connection. Contains the current state of the download queue and history.
- `subscriptions_all`: Emitted on connection or when all subscriptions are re-synced.
- `configuration`: Emitted on connection. Contains frontend-safe server settings.
- `custom_dirs`: Emitted on connection if `CUSTOM_DIRS` is enabled. Lists available subdirectories.
- `ytdl_options_changed`: Emitted when the custom yt-dlp options file is modified.
- `added`: Emitted when a new download is added.
- `updated`: Emitted when a download's progress or status changes.
- `completed`: Emitted when a download finishes successfully.
- `canceled`: Emitted when a download is canceled.
- `cleared`: Emitted when a download is removed from the history.
- `subscription_added`: Emitted when a new subscription is created.
- `subscription_updated`: Emitted when a subscription is updated or checked.
- `subscription_removed`: Emitted when a subscription is deleted.

### Client to Server (Events)

MeTube's Socket.IO interface is primarily for server-to-client updates. All actions (adding downloads, etc.) should be performed via the REST API.
