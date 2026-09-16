# Grabinator

[![Tests](https://github.com/<you>/grabinator/actions/workflows/test.yml/badge.svg)](https://github.com/<you>/grabinator/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)
![Platforms](https://img.shields.io/badge/platform-Linux%20%7C%20Windows%20%7C%20macOS-lightgrey)

A command-line downloader for **TikTok, YouTube, Dailymotion, SoundCloud, Instagram,
X, and Threads** — security-hardened, automatic
quality-aware deduplication, and playback fixes for native macOS players. Also
converts local video files to MP3 with `--convert`, no network access required.

## Table of contents

- [Features](#features)
- [Supported platforms (OS)](#supported-platforms-os)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Security](#security)
- [Legal](#legal)
- [Development](#development)
- [Author](#author)
- [License](#license)

## Features

| | |
|---|---|
| **Sources** | TikTok, YouTube, Dailymotion, SoundCloud, Instagram, X, Threads |
| **Naming** | TikTok/Instagram/X/Threads → `@username_id.mp4` · YouTube/Dailymotion → `title.mp4` |
| **Organization** | Each platform gets its own subfolder (`TikTok/`, `YouTube/`, etc.) |
| **Audio** | `--mp3` extracts clean, padding-free audio on any platform |
| **Local conversion** | `--convert PATH` turns existing video files into MP3s — no downloading |
| **Playlists** | Full YouTube/SoundCloud playlist support, with `--range` to grab a specific span |
| **Quality** | Capped one tier above 1080p by default, or pick interactively with `-q` |
| **Dedupe** | Re-downloading never creates duplicates — a lower/equal-quality repeat is skipped, a genuinely better one replaces the old file |
| **macOS fix** | Auto re-encodes tracks that play in VLC but are silent/blank in QuickTime, Preview, or Photos |
| **Privacy** | `--proxy` routes every request (including the connectivity check) through a SOCKS5/HTTP(S) proxy |
| **Size check** | `-c` shows the estimated download size — one total for a whole playlist — before anything downloads |

## Supported platforms (OS)

Pure Python, using only `pathlib`/`subprocess` for filesystem and process work — it
runs the same way on:

- **Linux** — any distro with Python 3.10+ and `ffmpeg` available.
- **macOS** — including the playback-compatibility fixes this tool specifically adds.
- **Windows** — the script forces UTF-8 console output and enables ANSI color
  processing on startup, so the colored status output and emoji indicators render
  correctly instead of crashing or printing garbled escape codes on a legacy `cmd.exe`
  codepage. Works from PowerShell, Windows Terminal, or plain `cmd.exe`.

The one external dependency that varies by OS is `ffmpeg` itself — see
[Installation](#installation) for the install command on each.

## Installation

```bash
git clone https://github.com/<you>/grabinator.git
cd grabinator
pip install -e .
```

This also installs `yt-dlp`, `tqdm`, and `certifi`. Separately, install `ffmpeg`
(which includes `ffprobe`) via your OS package manager:

```bash
brew install ffmpeg          # macOS
sudo apt install ffmpeg      # Debian/Ubuntu
winget install ffmpeg        # Windows
```

## Usage

### Single items

```bash
grabinator "https://www.tiktok.com/@user/video/123"
grabinator "https://www.youtube.com/watch?v=XXXXXXXXXXX"
grabinator "https://www.dailymotion.com/video/XXXXXXX"
grabinator "https://soundcloud.com/artist/track-name"
grabinator "https://www.instagram.com/p/XXXXXXXXXXX/"
grabinator "https://x.com/user/status/XXXXXXXXXXX"
grabinator "https://www.threads.net/@user/post/XXXXXXXXXXX"
```

### Playlists

```bash
grabinator "https://www.youtube.com/playlist?list=XXXXXXXXXXX"
grabinator "https://soundcloud.com/artist/sets/album-name"

# Only videos 3 through 7 of a playlist
grabinator "PLAYLIST_URL" --range 3-7

# Just the first 10
grabinator "PLAYLIST_URL" --range 10
```

### Multiple URLs at once

```bash
grabinator "URL1,URL2,URL3"
```

### Audio

```bash
# Audio only, on any platform — clean MP3, no leading silence
grabinator "URL" --mp3

# Convert local video files to MP3 instead — no network access at all,
# originals are never touched. PATH can be a file, a folder, or a .txt
# manifest listing one path per line.
grabinator --convert /path/to/videos
grabinator --convert /path/to/video.mp4
grabinator --convert /path/to/list.txt
```

### Quality control

```bash
# Interactively pick a resolution from a numbered menu
grabinator "URL" -q

# See the estimated file size and confirm before downloading
# (one total for an entire playlist, not one prompt per video)
grabinator "URL" -c
```

### Networking

```bash
grabinator "URL" --proxy socks5://127.0.0.1:9050
grabinator "URL" --proxy http://user:pass@host:port
```

### Other flags

```bash
grabinator "URL" --silent                  # suppress progress bars and status output
grabinator "URL" --output-dir /some/path   # override the default download folder
grabinator --version                       # print the installed version
```

Run without installing, straight from the source file:

```bash
python src/grabinator/cli.py "URL"
```

## Configuration

The default download folder is set at the top of `src/grabinator/cli.py`:

```python
OUTPUT_DIR = Path.home() / "Downloads" / "Grabinator"
```

Edit that constant, or pass `--output-dir` on the command line to override it per run.
Whatever you choose, each platform still gets its own subfolder underneath it.

## Security

- URLs are checked against an explicit host allowlist (TikTok/YouTube/Dailymotion/
  SoundCloud/Instagram/X/Threads domains only) before any network request is made.
- All subprocess calls (`ffmpeg`, `ffprobe`) use argument lists, never `shell=True`.
- Filenames derived from remote titles/usernames are sanitized, and every final
  destination path is resolved and verified to stay inside the configured output
  directory before anything is written — blocking path traversal from a malicious or
  unexpected title.
- Per-run URL and playlist-size caps guard against accidental bulk-scraping.
- The connectivity check only probes the exact host a given URL points to — never a
  fixed external address unrelated to what you asked to download.
- `--convert` never opens the network at all, and never modifies or deletes the
  original video files it reads.

## Legal

Built on [yt-dlp](https://github.com/yt-dlp/yt-dlp) and `ffmpeg`. Use it for content
you have the right to download — your own uploads, permissively licensed content, or
personal archival use consistent with each platform's Terms of Service and applicable
copyright law. You're responsible for how you use it.

## Development

```bash
pip install -e ".[dev]"
pytest
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a new platform, what's expected
of a pull request, and the release process. See [CHANGELOG.md](CHANGELOG.md) for the
project's history.

## Author

EagleSquwak

## License

[MIT](LICENSE)
