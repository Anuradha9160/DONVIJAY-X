# CDML-X — VPS Deployment & Media Tools

**Release:** `v3.1.12-x`

CDML-X is a Telegram mirror/leech bot for Linux VPS deployments. This release streamlines the Video Tools (`-vt`) menu and upgrades the Audio & Subtitle Track Manager (`-tm`).

## Media tools

### `-vt` — Video Tools Studio

The `-vt` selector is focused on video operations and synchronization:

- **Video + Video:** uses the same merge/planner workflow as `-ht`, with Normal or Advanced mode.
- **SubSync:** manually or automatically synchronizes subtitle files.
- **Audio Sync:** synchronizes an external audio track with the video.
- **Compress, Convert, Watermark, Trim, Extract, Remove Ads & Names, Media Split → Binary, Rename.**
- **MULTI VT:** runs supported operations sequentially, passing each result to the next selected tool.

Owner/sudo restrictions remain enforced for **Compress, Convert, Watermark, SubSync, and Audio Sync**. Processing status identifies the active tool, for example `Video + Video (Normal)`, `SubSync (Manual)`, `Audio Sync`, `Compress`, or `Watermark`, instead of showing a generic FFmpeg status for every Video Tools action. The status keeps the shared FFmpeg progress bar and ETA; Track Manager remuxing is labeled `Track Manager (Remux)`.

Audio/subtitle track editing is handled by `-tm` rather than cluttering the `-vt` selector. The subtitle translation and Whisper audio-to-subtitle features were removed, including their optional Python dependencies.

### `-tm` — Audio & Subtitle Track Manager

Use `-tm`, `-tr`, `-track`, or `-trackmanager` on a mirror/leech command, for example:

```bash
/mirror https://example.com/video.mkv -tm
```

The interactive manager supports:

- Selecting or removing audio and subtitle streams.
- Reordering audio and subtitle streams independently.
- **Select All / Select None** shortcuts for either stream type.
- Editing each track's language code and display title. Press **Edit** next to a track, then send `language | title`, such as `eng | English Dub` or `tel | Telugu AAC`. Use a 2–3 letter language code and leave the title blank to clear it.
- Setting the default audio/subtitle track (only one default per stream type) and marking subtitle tracks as **Forced**.
- Applying selection/order rules across multiple files in a batch. **Apply to All** matches tracks by detected language and reuses selection/order, language/title edits, and default/forced choices when matching tracks exist. Unmatched tracks are retained at the end but left unselected.
- Preserving the source video, chapters, global metadata, and existing disposition flags while stream changes are remuxed without re-encoding.

Use an ISO-style three-letter language code where possible (`eng`, `tel`, `jpn`, `und`). Track metadata changes only take effect when you press **Done**. Use **Cancel** to leave the file unchanged.

## Verification, shorteners, and metadata settings

### Verification controls

- `/verify` requests a verification link. In a group, use the **Send verification link to DM** button; the bot sends the actual link privately.
- `/verify on [duration]` enables verification. Durations accept seconds or readable values such as `30m`, `12h`, `7d`, `2w`, `1mo`, or `1y`.
- `/verify off` disables the verification gate.
- `/verify time 7d` changes the default authorization period; `/verify time never` enables unlimited validity until revoked.
- `/verify user USER_ID 7d` manually authorizes a user for a period; `/verify user USER_ID revoke` revokes that authorization.
- `/verify status` shows the current toggle, validity, and configured shortener count. `/start` remains available when verification is enabled.

### Shortener management

Use `/shorteners` as the owner/sudo admin to add, replace, or delete domains. Domain and API-key input is handled in the admin's DM. Configured shorteners are selected randomly for verification links and persisted to `shortener.txt` and the database when configured. Configure each shortener according to its provider's documented API format.

### Metadata settings

In User Settings, a single plain value such as `My Movie` is accepted as the `title` metadata and is applied to video, audio, and subtitle streams. Use `title=My Movie|artist=Studio` for multiple tags, and `\|` when a literal pipe belongs inside a value. Supported placeholders include `{filename}`, `{basename}`, `{extension}`, `{year}`, `{season}`, `{episode}`, `{resolution}`, `{width}`, `{height}`, `{video_codec}`, `{audio_codec}`, `{channels}`, `{channel_layout}`, `{duration}`, `{audiolang}`, and `{sublang}`. Unknown placeholders are preserved instead of crashing a task. Per-stream settings override matching keys from general metadata settings.

All command handlers accept uppercase or lowercase command names, and `CMD_SUFFIX` is added as an accepted variant for base and plugin commands.

## One-click deployment (Ubuntu/Debian VPS)

### 1. Get the project

```bash
cd ~
git clone https://github.com/hemanthbreaker3-hash/CDML-X.git CDML-X
cd CDML-X
```

If you have extracted the ZIP already, run these commands from that extracted project directory instead.

### 2. Configure credentials

Set these environment variables or enter the same values in `config.py`:

```bash
export BOT_TOKEN='REPLACE_WITH_BOT_TOKEN'
export TELEGRAM_API='REPLACE_WITH_TELEGRAM_API_ID'
export TELEGRAM_HASH='REPLACE_WITH_TELEGRAM_API_HASH'
export OWNER_ID='YOUR_TELEGRAM_USER_ID'
export DATABASE_URL='REPLACE_WITH_MONGODB_URI'
export CMD_SUFFIX='a'
```

Optional start-menu buttons can be configured with `START_REPO_URL`, `START_REPO_BUTTON`, `START_UPDATE_URL`, `START_UPDATE_BUTTON`, `START_DEV_URL`, and `START_DEV_BUTTON`.

### 3. Install and start

```bash
chmod +x one-click-deploy.sh deploy.vps deploy-user.vps start.sh
./one-click-deploy.sh
```

Run as root for system packages and systemd integration. When launched by a non-root user, the wrapper requests authorized sudo; if sudo is unavailable, it falls back to user-local mode. User-local mode cannot install system packages or create system services.

The installer checks Python syntax before installing dependencies, sets up FFmpeg and other media utilities, configures Deno as the yt-dlp JavaScript runtime, and installs the local YouTube PO-token provider. There is no interactive Deno PATH prompt.

## Operations

### User-local deployment

```bash
./run-user.sh                 # foreground start, after deployment
tail -f logs/user-bot.log    # inspect the deployment-launched process
kill "$(cat .cdmlx-user.pid)" # stop the deployment-launched process
```

### Root/systemd deployment

```bash
systemctl status cantarellabots_bot --no-pager -l
journalctl -u cantarellabots_bot -f
systemctl status bgutil-ytdlp-pot-provider --no-pager -l
journalctl -u bgutil-ytdlp-pot-provider -f
```

## Troubleshooting

- **Missing deploy script:** run commands from the extracted `CDML-X-main` project directory; do not run them from your home directory unless the scripts are there.
- **Missing media tools:** verify `ffmpeg`, `ffprobe`, `mediainfo`, `7z`, and `aria2` are installed. User-local mode cannot install missing OS packages.
- **YouTube extraction challenges:** verify the bot's cookies/settings and check both `cantarellabots_bot` and `bgutil-ytdlp-pot-provider` service logs.
- **Track Manager says no streams found:** the selected file must contain an audio or subtitle stream. A video-only file has no track options to edit.
- **No usable output after a Video Tool:** inspect the task log for the named tool's FFmpeg error, confirm the source media is readable, and check available disk space.

## Security notes

Keep `.cdmlx-user.env`, `config.py`, session strings, bot tokens, and database credentials private. Revoke and regenerate credentials if they are exposed. The deployment helper does not bypass sudo policy or grant additional system permissions.

## Release notes (`v3.1.12-x`)

- Added runtime verification controls, per-user authorization validity, and multi-shortener administration.
- Applied plain-value metadata settings to all stream types and added safe filename/media placeholders.
- Made command aliases case-insensitive and accepted `CMD_SUFFIX` variants across commands.
- Reworked the `-vt` selector to focus on video operations while retaining SubSync and Audio Sync.
- Expanded `-tm` with per-track language/title edits and bulk selection controls.
- Added operation-specific status labels throughout Video Tools processing.
- Removed subtitle translation and audio-to-subtitle transcription code and dependencies.
- Updated plugin minimum bot-version metadata and refreshed deployment/documentation guidance.
