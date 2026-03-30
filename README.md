# Vimeo Account Video Downloader

A Python desktop application (Tkinter GUI) that downloads videos from any Vimeo user profile using a credentials JSON file.

---

## Features

- **Credentials loaded from a JSON file** — browse for a file containing `access_token`, `client_id`, and `client_secret`; all three fields are displayed as masked entries with individual Show/Hide toggles
- **Client-credentials OAuth flow** — if no `access_token` is provided, the app automatically exchanges `client_id` + `client_secret` for a token via Vimeo's OAuth endpoint
- **Configurable profile URL** — download from any public Vimeo profile (`https://vimeo.com/username`), channel (`https://vimeo.com/channels/name`), or leave blank to use the authenticated account (`/me/videos`)
- **Fetch limit** — optionally cap how many videos are fetched (useful for quick testing); leave empty to fetch all
- Fetches the **complete video library** with automatic pagination
- Displays videos in a scrollable, interactive checklist with:
  - **Video Title** — original title from Vimeo
  - **Video File Name** — auto-generated sanitized filename (see [File Naming](#file-naming) below)
  - **Stream Date** — actual livestream date/time from the video file metadata (more accurate than upload date), converted to local timezone with timezone label
  - **Video ID** — unique Vimeo video identifier (clickable via double-click to open in browser)
  - **Duration**
  - **Best available quality**
  - **File size**
  - **Download status** — color-coded: blue (Downloading), green (Done ✓), red (Failed), yellow (Skipped)
- **Filter toggle** — hide videos that have no duration, quality, or size information
- **Two-stage download strategy**
  1. Direct signed URL from the Vimeo API (fastest, no extra tool required)
  2. Falls back to **yt-dlp** for videos without an API download link
- Quality selector: Best Available / HD / SD / Mobile
- Per-video **progress bar** with bytes downloaded and percentage
- Overall progress bar across all selected videos
- **Skip already-downloaded** files automatically (checks file size)
- **Pre-marked already-downloaded videos** — when Fetch is clicked, the app scans the output folder for existing `.mp4` files; any video whose generated filename already exists is shown as "Done ✓" (green) with its checkbox disabled and unchecked, so it is excluded from the next download batch
- Optional **number-prefix filenames** (e.g. `001_20240315_143022_Sunday_Service.mp4`)
- Scrollable main window — all controls remain accessible at any window size
- Cancel in-progress downloads cleanly
- Timestamped log panel with a **Clear Log** button
- **Total estimated size** displayed in the log after fetching (counts unknown-size videos separately)
- **Logged-in user** label shown next to the Fetch button after successful authentication
- **GitHub README link** — clickable link in the Configuration section to view full documentation in your browser

---

---

## Quick Start

### Prerequisites
- **Python 3.10 or later** — [Download here](https://www.python.org/downloads/)
- **pip** — comes with Python 3.10+

### Step 1: Clone or Download
Download this project to your machine.

### Step 2: Install Dependencies
Open a terminal in the project folder and run:

```bash
pip install -r requirements.txt
```

### Step 3: Create Credentials File
1. Follow the [Credentials JSON File](#credentials-json-file) section below to generate your Vimeo API credentials
2. Save them in a JSON file (e.g., `vimeo_creds.json`)

### Step 4: Run the Application
```bash
python app.py
```

The GUI will open. Click **Browse** to select your credentials file, then **Fetch Videos** to start.

---

## Requirements

- **Python 3.10 or later**
- `PyVimeo` — official Vimeo Python SDK for all API calls
- `requests` — HTTP client (used internally by PyVimeo and for streaming downloads)
- `yt-dlp` — fallback downloader for videos without a direct API link

All dependencies are installed via:

```bash
pip install -r requirements.txt
```

---

## Credentials JSON File

Create a JSON file with the following structure:

```json
{
  "access_token": "your_personal_access_token",
  "client_id": "your_client_id",
  "client_secret": "your_client_secret"
}
```

All three keys are required. You can omit the token value (leave it as an empty string `""`) to have the app obtain one automatically using the client credentials.

### Where to get these values

1. Sign in at <https://developer.vimeo.com/> with the account that owns the videos.
2. Go to **My Apps** → **Create an app** (or open an existing app).
3. Copy the **Client Identifier** and **Client Secret** from the app page.
4. Under **Authentication**, generate a **Personal Access Token** with the scopes: `public`, `private`, `video_files`.
5. Save all three values into your credentials JSON file.

> **Tip:** The `video_files` scope is required for direct download links. Without it the app will fall back to yt-dlp for every video.

---

## File Naming

Downloaded files are named automatically using the following format:

```
YYYYMMDD_HHMMSS_<sanitized_title>.mp4
```

- **`YYYYMMDD_HHMMSS`** — the video's creation timestamp in your local timezone
- The title is sanitized by replacing spaces, periods, and any Windows-invalid characters (`\ / : * ? " < > |`) with `_`
- The first letter after each `_` separator is capitalised

**Examples:**

| Vimeo Title | Generated filename |
|---|---|
| `Sunday Service 03.09.25` | `20250309_103000_Sunday_Service_03_09_25.mp4` |
| `Weekly Update: Team/Dev` | `20260114_090000_Weekly_Update__Team_Dev.mp4` |

---

## How to Run

### Option 1: Python Script (Development)
```bash
python app.py
```

Requires Python 3.10+ and dependencies installed (`pip install -r requirements.txt`).

### Option 2: Standalone Executable (No Python Required)
Download the pre-built `.exe` from the [Releases](../../releases) page, or build your own following the [Building a Standalone Executable](#building-a-standalone-executable) section below.

---

## Building a Standalone Executable

Convert the Python script into a Windows `.exe` file that doesn't require Python to be installed.

### Quick Build
```bash
pip install -r requirements-dev.txt
python build.py
```

This generates `dist/CBCMVimeoDownloader-v-0-0-1.exe` (single-file mode).

### Specify Version Number
```bash
python build.py 1.0.0
```

Creates `dist/CBCMVimeoDownloader-v-1-0-0.exe` with the version updated throughout the app.

### Windows Compatibility Mode (Recommended)
If you get startup warnings on Windows, use folder mode for better compatibility:

```bash
python build.py 1.0.0 --onedir
```

This creates a folder with the executable and all dependencies, with faster startup and better antivirus compatibility.

For full details, see [BUILD.md](BUILD.md).

---

## Troubleshooting

### Issue: "ModuleNotFoundError: No module named 'vimeo'"
**Solution:** Install dependencies:
```bash
pip install -r requirements.txt
```

### Issue: "Credentials JSON file not found"
**Solution:** Click **Browse** in the GUI to select your credentials file. Ensure the file contains valid JSON with `access_token`, `client_id`, and `client_secret` keys.

### Issue: Authentication fails ("401 Unauthorized")
**Possible causes:**
- Expired or invalid credentials — regenerate in your Vimeo app settings
- Missing `video_files` scope on token — the download link scope is required
- Incorrect JSON format — validate your credentials file is valid JSON

**Solution:** Re-create your credentials file following the [Credentials JSON File](#credentials-json-file) section.

### Issue: Videos are marked as "Failed" (red)
**Possible causes:**
- Video privacy settings prevent download (e.g., privacy-restricted)
- Network interruption during download
- yt-dlp installation issue (fallback mechanism)

**Solution:**
- Check the log panel for detailed error messages
- Ensure all videos are public or you have permission
- Re-run the download for those specific videos

### Issue: yt-dlp command not found
**Solution:** Install yt-dlp:
```bash
pip install yt-dlp
```

### Issue: Slow startup on Windows
**Solution:** If you built a single-file executable, try the folder mode:
```bash
python build.py 1.0.0 --onedir
```

---

## Project Information

**Status:** Active Development  
**License:** See [LICENSE](LICENSE) file (if present)  
**Python Version:** 3.10+  
**Last Updated:** March 2026

For more details on configuration, building, and advanced usage, see:
- [BUILD.md](BUILD.md) — Building standalone executables
- [REFACTORING.md](REFACTORING.md) — Code improvements and technical notes

The **Video File Name** column in the video list shows the exact filename that will be used before you start a download.

If the **Number-prefix filenames** option is enabled, a zero-padded index is prepended:

```
001_20240315_143022_Sunday_Service.mp4
```

---

## Usage

```bash
python app.py
```

1. Click **Browse…** next to **Credentials File** and select your credentials JSON.
2. The Access Token, Client ID, and Client Secret fields will populate automatically (masked by default — click **Show** to reveal any field).
3. Optionally edit the **Profile URL** to target a specific Vimeo profile or channel (defaults to `https://vimeo.com/cbcmrcf`).
4. Choose an **Output Folder** (defaults to `~/Downloads/Vimeo`).
5. Optionally enter a **Limit** to fetch only the first N videos (leave empty to fetch all).
6. Select a **Quality** preference and click **🔍 Fetch Videos**.
7. Once the list loads, use **Select All / Deselect All** or click individual checkboxes to choose which videos to download. Videos whose generated filename already exists in the output folder are pre-marked as **Done ✓** and cannot be re-selected. Review the **Video File Name** column to confirm the generated filenames.
   - **Single-click** the checkbox to toggle a video's selection
   - **Double-click** any row to open that video in your default browser
8. Optionally enable **Hide videos without duration / quality / size** to filter out unavailable videos.
9. Click **⬇ Download Selected** and monitor progress in real time.
10. Click **Open Folder** to open the output directory when done.
11. Click **Clear Log** to clear the log panel at any time.

### Stream Date vs Last Modified

The **Stream Date** column shows the actual creation/broadcast timestamp from the video file metadata (extracted from the Vimeo download object), which is more accurate than the upload date. This is particularly useful for livestream archives to see when the stream actually occurred. The timestamp is displayed in your local timezone (e.g. `2026-03-08 14:30 (EST)`).

---

## Building an Executable

To create a standalone Windows EXE or other platform executables, use the build script:

```bash
python build.py
```

This will generate a single-file executable in the `dist/` folder with a default development version (`0.0.1`).

### Building a Release Version

Specify a version number for a release build:

```bash
python build.py 1.0.2
```

This will:
- Update the app version to `1.0.2`
- Create `dist/CBCMVimeoDownloader-v-1-0-2.exe` (or equivalent for other platforms)
- Automatically clean up temporary build artifacts (`build/` folder and build.spec file)
- Display the final executable location and file size

For more details, see [BUILD.md](BUILD.md).

---

## Project Structure

```
├── app.py            # Main GUI application
├── build.py          # Cross-platform build script
├── requirements.txt  # Python dependencies
├── BUILD.md          # Build instructions
└── README.md
```

---

## Security Notes

- All credential fields are masked in the UI by default; each has an individual **Show** toggle.
- Credentials are never written to disk by this application — they are loaded read-only from the file you provide.
- All API communication is over HTTPS to `api.vimeo.com` only.
- The Bearer token is sent only in the `Authorization` header of API and download requests.
