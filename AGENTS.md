
# 🤖 AGENTS.md - YouTube Archiver Agent

## 📌 Overview
This document outlines the behavior and architecture of the **Automated YouTube Archiving Agent**. 
Currently running inside a Google Colab environment, this agent is designed to autonomously monitor a specific list of YouTube channels, identify new uploads, download them at high quality, and securely back them up as GitHub Releases.

---

## ⚙️ How It Works (The Workflow)
When the Colab notebook is executed, the agent performs the following step-by-step pipeline:

1. **Environment Setup & Authentication:**
   - Retrieves the GitHub Personal Access Token (`GH_TOKEN`) from Colab's secure secrets.
   - Configures the `gh` (GitHub CLI) and standard `git` credentials.
   - Clones the target repository (`AliMohammadiCoder/yt`) into the Colab workspace.

2. **Discovery & Filtering:**
   - Reads the `watch_list.txt` file to get the list of target YouTube channels.
   - Automatically appends `/videos` to each channel URL. **(Rule: Strict exclusion of "Shorts" and "Live" streams).**
   - Scans only the **latest 5 videos** from the standard videos tab to optimize speed and resource usage.

3. **State Management (The Brain):**
   - The agent consults its internal database, `archived.txt`.
   - Before attempting any downloads, `yt-dlp` checks if the video ID exists in this file. If it does, the video is instantly skipped.

4. **Downloading:**
   - Downloads new videos at a maximum resolution of **1080p** (merging best video and best audio).
   - Dynamically renames files to strip out special characters/spaces and applies the format: `[Uploader]_[Title]_[Upload_Date].[ext]`.
   - Skips deleted, private, or unavailable videos gracefully without crashing the pipeline.

5. **Cloud Backup & Release:**
   - **Database Update:** The agent updates `archived.txt` with the new video IDs, commits the changes, and pushes them back to the `main` branch of the GitHub repository.
   - **Asset Upload:** Generates a dynamic timestamped tag (e.g., `v2026.05.03-143000`) and uses the GitHub CLI to upload all newly downloaded video files as a GitHub Release.

6. **Cleanup:**
   - Deletes all local video files from the Colab environment to prevent running out of disk space.

---

## 📂 Key Files
The agent relies on the following file structure within the repository:

* **`watch_list.txt`**: A user-managed text file containing the URLs of the YouTube channels to monitor (one URL per line).
* **`archived.txt`**: A machine-managed database file where `yt-dlp` logs the IDs of successfully downloaded videos (e.g., `youtube dQw4w9WgXcQ`). **Do not manually edit this unless you want to force a re-download.**
* **The Colab Notebook**: The actual Python execution environment running the logic.

---

## 🛑 Current Rules & Limitations (At This Stage)
* **Depth Limit:** Only checks the most recent **5** uploads per channel.
* **Content Filter:** Only downloads standard VODs. **No Shorts. No Live Streams.**
* **Resolution Limit:** Capped at `1080p` to balance visual quality and GitHub Release file size limits.
* **Trigger Mechanism:** Currently requires a manual run of the Google Colab notebook (not yet scheduled via cronjobs or GitHub Actions).

---















# 🤖 AGENTS.md - YouTube Archiver Agent (VPS Edition)

## 📌 Overview
This document outlines the behavior and architecture of the **Automated YouTube Archiving Agent**. Optimized for 24/7 execution on a VPS within a JupyterLab environment, this agent autonomously monitors a channel watch list and processes manual "on-demand" video requests, backing everything up to GitHub Releases.

---

## ⚙️ How It Works (The Dual-Mode Pipeline)
The agent operates in a continuous loop (30-minute intervals), performing the following logic:

### 1. Mode A: On-Demand Requests (Priority)
- **Source:** Scans `requestedvids.txt`.
- **Bypass Logic:** To maximize speed and minimize resource usage, these videos **bypass `yt-dlp` metadata extraction**.
- **Extraction:** Uses localized Regex to pull the 11-character Video ID directly from the URL.
- **State:** These videos are **not** logged in `archived.txt`, allowing for repeated downloads if requested again.

### 2. Mode B: Watch List Sync (Discovery)
- **Source:** Reads `watch_list.txt`.
- **Discovery:** Uses `yt-dlp` in "Flat Extraction" mode (metadata only) to check the latest **3** uploads per channel.
- **Filtering:** 
    - **Archive Check:** Consults `archived.txt`. If the ID exists, it is skipped.
    - **Duration Check:** Skips any video longer than **2 hours** (7200s) to prevent API timeouts and massive file sizes.
    - **Content Rule:** Strictly targets the `/videos` tab (skips Shorts and Live streams).

### 3. Converting & Downloading (The API Bridge)
- Instead of downloading directly (which often triggers YouTube rate limits on VPS IPs), the agent uses the **`hub.ytconvert.org` API**.
- **Process:**
    1. Sends a conversion request (720p/1080p MP4).
    2. Polls a status URL until the conversion is "completed."
    3. Streams the file directly from the conversion server to the VPS disk.
- **Mid-Stream Safety:** If a file exceeds **1.95 GB** during the download, the agent aborts and deletes the partial file to ensure it fits within GitHub's 2GB Release limit.

### 4. State Synchronization (Self-Healing Git)
The agent manages its own repository state to ensure the VPS and GitHub never fall out of sync:
- **Auto-Clear:** Once `requestedvids.txt` is processed, it is wiped clean.
- **Conflict Resolution:** Uses `git pull --rebase --autostash`. If you edit the `watch_list.txt` on GitHub Web while the script is running, the agent will automatically merge those changes without crashing or requiring manual intervention.
- **Commit Logic:** Updates `archived.txt` with new IDs and pushes the cleared `requestedvids.txt` back to the `main` branch.

### 5. Cloud Backup & Cleanup
- **Asset Upload:** All successful downloads are bundled into a timestamped GitHub Release (e.g., `v2026.05.06-123000`).
- **Disk Management:** Immediately deletes all `.mp4`, `.mkv`, and `.webm` files after the upload (or if they are rejected for size) to prevent the VPS disk from filling up.

---

## 📂 Key Files
*   **`watch_list.txt`**: List of channel URLs to monitor (User-managed).
*   **`requestedvids.txt`**: List of specific video URLs to download immediately (User-managed).
*   **`archived.txt`**: Database of previously downloaded watchlist videos (Machine-managed).
*   **`archiver.log`**: Timestamped summary of every 30-minute cycle.

---

## 🛑 Logic Rules & Constraints
*   **Release Limit:** Strictly **2.00 GB** per file (capped at 1.95 GB for safety).
*   **Duration Limit:** Strictly **2 hours** per video.
*   **Retry Logic:** 5 retries per video if the API returns an internal error.
*   **Jupyter Optimization:** Uses `clear_output` every cycle to keep the browser tab responsive and prevent memory leaks in the Jupyter UI.

---

## 🛠️ Dependencies
- `yt-dlp`: Used exclusively for flat metadata discovery.
- `requests`: Used for API communication and file streaming.
- `gh` CLI: Used for managing GitHub Releases.
- `git`: Used for state management and database syncing.

## 🛠️ Required Dependencies
- `yt-dlp` (Video processing and extraction)
- `gh` CLI (GitHub releases management)
- `nodejs` (To resolve YouTube extraction JavaScript runtime requirements)
