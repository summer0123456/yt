
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

## 🛠️ Required Dependencies
- `yt-dlp` (Video processing and extraction)
- `gh` CLI (GitHub releases management)
- `nodejs` (To resolve YouTube extraction JavaScript runtime requirements)
