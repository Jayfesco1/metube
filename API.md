# MeTube REST API Documentation

All endpoints are relative to the `URL_PREFIX` (default: `/`).

## General Endpoints

### `GET /version`
Returns the current version of MeTube and the underlying `yt-dlp`.
- **Response**: `{"yt-dlp": "...", "version": "..."}`

### `GET /presets`
Returns the list of configured `yt-dlp` option presets.
- **Response**: `{"presets": ["preset1", "preset2", ...]}`

### `GET /history`
Returns the history of downloads from disk. **Note**: This endpoint returns persisted data and does *not* include real-time fields like `percent`, `speed`, or `eta`. Use Socket.IO for real-time progress.
- **Response**:
  ```json
  {
    "done": [DownloadObject, ...],
    "queue": [DownloadObject, ...],
    "pending": [DownloadObject, ...]
  }
  ```

### `GET /cookie-status`
Checks if cookies are currently configured (either uploaded or via environment).
- **Response**: `{"status": "ok", "has_cookies": true/false}`

---

## Download Endpoints

### `POST /add`
Adds a new download to the queue.
- **Request Body**:
  ```json
  {
    "url": "https://...",
    "download_type": "video" | "audio" | "captions" | "thumbnail",
    "quality": "best" | "2160" | "1080" | ... | "worst",
    "format": "any" | "mp4" | "mp3" | ...,
    "folder": "optional/subfolder",
    "custom_name_prefix": "optional_prefix",
    "playlist_item_limit": 0,
    "auto_start": true,
    "split_by_chapters": false,
    "chapter_template": "%(title)s - %(section_number)02d - %(section_title)s.%(ext)s",
    "subtitle_language": "en",
    "subtitle_mode": "prefer_manual" | "auto_only" | "manual_only" | "prefer_auto",
    "ytdl_options_presets": ["preset1"],
    "ytdl_options_overrides": {}
  }
  ```
- **Response**: `{"status": "ok", "id": "url_or_id"}` or `{"status": "error", "msg": "..."}`

### `POST /cancel-add`
Cancels a long-running playlist or channel expansion operation triggered by `/add`.
- **Response**: `{"status": "ok"}`

### `POST /delete`
Cancels active downloads or clears completed downloads from the list.
- **Request Body**:
  ```json
  {
    "ids": ["url1", "url2", ...],
    "where": "queue" | "done"
  }
  ```
- **Response**: `{"status": "ok"}`

### `POST /start`
Starts downloads that were added with `auto_start: false` (pending).
- **Request Body**:
  ```json
  {
    "ids": ["url1", "url2", ...]
  }
  ```
- **Response**: `{"status": "ok"}`

---

## Subscription Endpoints

### `GET /subscriptions`
Returns a list of all current subscriptions.
- **Response**: `[SubscriptionObject, ...]`

### `POST /subscribe`
Creates a new subscription.
- **Request Body**: Same as `POST /add`, with an additional optional field:
  ```json
  {
    "check_interval_minutes": 60
  }
  ```
- **Response**: `{"status": "ok", "subscription": SubscriptionObject}` or `{"status": "error", "msg": "..."}`

### `POST /subscriptions/update`
Updates an existing subscription.
- **Request Body**:
  ```json
  {
    "id": "subscription-uuid",
    "name": "New Name",
    "enabled": true,
    "check_interval_minutes": 120
  }
  ```
- **Response**: `{"status": "ok", "subscription": SubscriptionObject}`

### `POST /subscriptions/delete`
Deletes one or more subscriptions.
- **Request Body**:
  ```json
  {
    "ids": ["id1", "id2", ...]
  }
  ```
- **Response**: `{"status": "ok"}`

### `POST /subscriptions/check`
Manually triggers a check for new items in subscriptions.
- **Request Body**:
  ```json
  {
    "ids": ["id1", "id2", ...] // Optional: if omitted, checks all enabled subscriptions
  }
  ```
- **Response**: `{"status": "ok"}`

---

## Real-time Updates (Socket.IO)

MeTube uses Socket.IO for real-time updates. The websocket endpoint is at `{URL_PREFIX}socket.io/`.

### Connection
Upon connection, the server emits several events to synchronize the client state:
- `all`: Returns the current in-memory state of the queue. Format: `[[ActiveAndPendingDownloads], [CompletedDownloads]]`.
- `subscriptions_all`: Returns all subscriptions.
- `configuration`: Returns the safe frontend configuration.
- `custom_dirs`: Returns available custom directories (if enabled).
- `ytdl_options_changed`: Emitted if the `YTDL_OPTIONS_FILE` is modified.

### Events
- `added`: A new download was added.
- `updated`: A download's progress or status changed (includes `percent`, `speed`, `eta`).
- `completed`: A download finished successfully.
- `canceled`: A download was canceled.
- `cleared`: A completed download was removed from the list.
- `subscription_added`: A new subscription was created.
- `subscription_updated`: A subscription was updated or its check finished.
- `subscription_removed`: A subscription was deleted.

---

## Downloading Files

MeTube serves downloaded files as static assets. The base URLs for these files are provided in the `configuration` Socket.IO event and via the `PUBLIC_HOST_URL` and `PUBLIC_HOST_AUDIO_URL` settings.

### Base URLs
- **Video/Other**: `{URL_PREFIX}download/` (Default)
- **Audio**: `{URL_PREFIX}audio_download/` (Default)

### Usage
To download a completed file, append the `filename` from a `DownloadObject` to the appropriate base URL:
`GET {URL_PREFIX}download/{filename}`

Example: if a download has `filename: "My Video.mp4"`, the download link is `{URL_PREFIX}download/My%20Video.mp4`.

---

## Cookie Management

### `POST /upload-cookies`
Uploads a `cookies.txt` file (Multipart form data).
- **Form Field**: `cookies` (file)
- **Response**: `{"status": "ok", "msg": "..."}`

### `POST /delete-cookies`
Deletes the uploaded `cookies.txt` file.
- **Response**: `{"status": "ok"}`

---

## Data Structures

### `DownloadObject`
```json
{
  "id": "...",
  "title": "...",
  "url": "...",
  "status": "pending" | "preparing" | "downloading" | "finished" | "error",
  "percent": 0.0,
  "speed": 0,
  "eta": 0,
  "filename": "...",
  "size": 0,
  "error": "...",
  "timestamp": 123456789,
  ...
}
```

### `SubscriptionObject`
```json
{
  "id": "...",
  "name": "...",
  "url": "...",
  "enabled": true,
  "check_interval_minutes": 60,
  "download_type": "video",
  "codec": "auto",
  "format": "any",
  "quality": "best",
  "folder": "...",
  "last_checked": 123456789,
  "seen_count": 0,
  "error": null
}
```
