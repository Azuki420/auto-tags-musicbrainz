# mp3tag CLI - Picard & MusicBrainz Automated Tagger

A CLI tool that downloads accurate metadata from the **MusicBrainz (Picard)** database, matches tracks intelligently based on folder name, writes tags, and standardizes audio filenames.

Compatible with **Puddletag**, **Mp3tag**, **MusicBrainz Picard**, and music servers/players like **Feishin**, **Navidrome**, and **Foobar2000**.

---

## Features

- **Folder-based Search**: Cleans folder names (stripping `[FLAC]`, `[320k]`, `(2023)`, etc.), parses `Artist - Album` or `Label - Album`, and finds the release on MusicBrainz.
- **Smart Candidate Scoring**: Ranks candidate releases by matching track counts, media/disc counts, title similarity, artist/label similarity, and release year.
- **Standardized File Renaming**:
  - **1 Disc Releases**:
    `01. BLACKPINK - BOOMBAYAH.flac`
  - **2 or More Discs**:
    `1.01. BABYMETAL - FUTURE METAL.flac`
- **Track & Disc Number Formatting**:
  - `track` / `tracknumber`: Always 2-digit zero-padded (`01`, `02`, `12`).
  - `discnumber` / `disc`: Never zero-padded (`1`, `2`, `10`).
- **Year Formatting**: Tags always use the 4-digit year (`2023`).
- **Catalog Numbers in Multi-Disc Albums**:
  - Resolves distinct catalog numbers for each disc (e.g. `IMAE-00046` on Disc 1, `IMAE-00047` on Disc 2) unless data provides only 1 catalog number.
- **Tag Names**:
  - All tag names are strictly lowercase (e.g. `album`, `artist`, `catalognumber`, `tracknumber`, `tracktotal`, `disctotal`).
- **Tag Sanitization**:
  - Clears all previous/junk tags (such as `encoder`, `replaygain_*`, `tsse`, etc.) to keep files pristine and clean.
- **Cover Art**:
  - Disabled by default.
  - When `--cover` is specified, downloads front cover art from the Cover Art Archive, verifies and converts to JPEG, embeds it into audio tags, and saves it as `albumart.jpg`.
- **Supported Formats**: `.flac`, `.mp3`, `.m4a`, `.mp4`, `.ogg`, `.opus`, `.wav`, `.aiff`.

---

## Installation & Setup

`mp3tag` is installed in `~/.local/bin/mp3tag` and added to `PATH` in `~/.zshrc` and `~/.bashrc`.

Verify it by running:
```bash
which mp3tag
# Output: /home/azuki/.local/bin/mp3tag
```

---

## Usage

### 1. Basic Usage (Interactive)
```bash
mp3tag some_folder
```
Example:
```bash
mp3tag "/mnt/tower/data/media/music/KONAMI - SOUND VOLTEX ULTIMATE TRACKS Dimension Gates Ω -Phase.1-"
```

The tool will:
1. Scan audio files and detect track/disc counts.
2. Search MusicBrainz for matching releases.
3. Show ranked candidate releases with scores.
4. Display a preview table of aligned tracks, **new filenames**, and **per-disc catalog numbers**.
5. Ask for confirmation before writing tags and renaming files.

### 2. Auto-Select Top Match (Fast / Non-Interactive)
Use `-y` or `--yes` to automatically select the top-scoring match, write tags, and rename files without prompting:
```bash
mp3tag some_folder -y
```

### 3. Dry-Run (Preview Only)
Simulate matching and preview all tag changes and file renames without modifying anything:
```bash
mp3tag some_folder -n
```

### 4. Enable Cover Art (albumart.jpg)
Cover art is disabled by default. Pass `--cover` to download, convert to JPEG, embed, and save `albumart.jpg`:
```bash
mp3tag some_folder --cover
```

### 5. Custom Search Query
If a folder name is unconventional or shortened, supply a custom query:
```bash
mp3tag some_folder -q "Steins Gate Original Soundtrack"
```

### 6. Direct MusicBrainz Release ID or URL
Directly tag from an exact MusicBrainz URL or UUID:
```bash
mp3tag some_folder --id "2c7418c5-763f-43ca-ae8c-862ae9fe1393"
```

---

## CLI Options

```text
usage: mp3tag [-h] [-y] [-n] [--cover] [-q QUERY] [--id ID] [-v] folder [folder ...]

positional arguments:
  folder             Directory containing the album or audio files

options:
  -h, --help         show this help message and exit
  -y, --yes          Non-interactive mode (auto-select top match & apply)
  -n, --dry-run      Preview tags and changes without writing to disk
  --cover            Download and embed front cover art (and save albumart.jpg). Default: disabled
  -q, --query QUERY  Override search query instead of using the folder name
  --id ID            MusicBrainz Release ID (UUID) or URL to fetch directly
  -v, --verbose      Enable verbose debug output
```
