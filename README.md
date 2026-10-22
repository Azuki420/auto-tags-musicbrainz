# mp3tag CLI - Picard & MusicBrainz Automated Tagger

A CLI tool that downloads accurate metadata from the **MusicBrainz (Picard)** database, matches tracks intelligently based on folder name, writes tags, and standardizes audio filenames.

Compatible with **Puddletag**, **Mp3tag**, **MusicBrainz Picard**, and music servers/players like **Feishin**, **Navidrome**, and **Foobar2000**.

---

## Features

- **Folder-based Search**: Cleans folder names (stripping `[FLAC]`, `[320k]`, `(2023)`, doujin/event prefixes like `[M3-56]`, `[C102]`, `[Reitaisai]`, etc.), parses `Artist - Album` or `Label - Album` (including irregular hyphen spacing), and finds the release on MusicBrainz.
- **Release Group & Release URL/MBID Support**: Accepts direct MusicBrainz `release` OR `release-group` URLs and MBIDs (via `--id`, interactive prompt, or manual query). When given a release group, it automatically matches and resolves the member releases against local tracks.
- **Smart Candidate Scoring**: Ranks candidate releases by matching track counts, media/disc counts, title similarity, artist/label similarity, and release year.
- **Multi-tier Query Cascading**: Uses album titles, subtitle segments, artist/circle hints, and embedded audio tag fallbacks to find releases even with complex multi-lingual naming.
- **Standardized File Renaming**:
  - **1 Disc Releases**:
    `01. BLACKPINK - BOOMBAYAH.flac`
  - **2 or More Discs**:
    `1.01. BABYMETAL - FUTURE METAL.flac`
- **Standardized Album Folder Renaming**:
  - Renames the album folder to:
    `{album artist} - {album name}/` (e.g. `Lady Gaga - The Fame Monster/`)
    or
    `{artist} - {album name}/` when album artist is not applicable.
- **Track & Disc Number Formatting**:
  - `TRACKNUMBER`: Always 2-digit zero-padded (`01`, `02`, `12`).
  - `DISCNUMBER`: Never zero-padded (`1`, `2`, `10`).
- **Year Formatting**: Tags always use the 4-digit year (`2023`).
- **Catalog Numbers in Multi-Disc Albums**:
  - Resolves distinct catalog numbers for each disc (e.g. `IMAE-00046` on Disc 1, `IMAE-00047` on Disc 2) unless data provides only 1 catalog number.
- **Tag Names**:
  - All tag names are strictly uppercase (e.g. `ALBUM`, `ARTIST`, `CATALOGNUMBER`, `TRACKNUMBER`, `TRACKTOTAL`, `DISCTOTAL`).
- **Tag Sanitization**:
  - Clears all previous/junk tags (such as `encoder`, `replaygain_*`, `tsse`, redundant `DISC`/`TRACK` aliases, and `musicbrainz_*` tags) to keep files pristine and clean.
- **Cover Art & Local Image Handling**:
  - Automatically searches the folder for existing album art or cover images (even with random filenames, `cover.png`, `folder.jpg`, etc.).
  - Converts any existing cover image to standard JPEG and renames it to `albumart.jpg` (removing duplicate/old filenames).
  - Audio tag embedding is disabled by default. Pass `--cover` to embed `albumart.jpg` into audio metadata (or download from Cover Art Archive if no local image is present).
- **LABEL Tag Sanitization**:
  - Genuine record labels are written to `LABEL`.
  - When no label is specified or MusicBrainz provides placeholder names (such as `[no label]`, `[none]`, `[unknown]`), the `LABEL` tag is left completely empty / omitted.
- **MEDIA Tag Detection**:
  - Automatically determined from folder content and name:
    - Files are 24-bit FLAC (or folder name indicates 24bit/WEB) -> `WEB`.
    - Folder contains `.cue` or `.log` files (or folder name indicates CD/EAC) -> `CD`.
    - When none of these are present, the `MEDIA` tag is left empty (omitted).
- **EVENT Tag Detection & Normalization**:
  - Detects event and convention codes from folder names (e.g. `[M3-56] ...`, `(C89) ...`, `[Reitaisai 21] ...`), existing audio tags, or MusicBrainz disambiguation.
  - Normalizes to canonical short codes:
    - Comiket (`C89`, `C109`, `CS6`, `Air Comiket`)
    - Music Media-Mix Market (`M3-50`, `M3-56`, `M3-Osaka`, `M3 Festa`)
    - Reitaisai (`Reitaisai 23`, `Fall Reitaisai 12`, `Reitaisai in Chichibu`, `Reitaisai SP2`)
    - Touhou Kouroumu (`Touhou Kouroumu 21`)
    - THE VOC@LOiD M@STER (`TVM61`)
    - Tokyo Game Show (`TGS 2025`)
    - Tokyo Game Music Show (`TGMS2025`)
    - Denki-guy Matsuri (`Denki-guy 2024W`, `Denki-guy 2013S`)
    - DizzyMart (`DM 02`)
    - SUPER RELEASE (`SR26`)
    - HOURAI-SAI (`HS4`)
    - Magical Mirai (`MM2025`)
    - Sunshine Creation (`SC46`), Comic Revolution (`CR35`), Comic Communication (`ComiComi 10`), BrightSeason (`BS12`), Heartful Communication (`HC`), RAG-FES, SDCC, and more.
  - Supports CLI override or specification via `-e` / `--event <name>`.
  - When no event is found or specified, the `EVENT` tag is left completely empty (omitted).
- **Auxiliary Files Handling (.m3u8, .cue, .log)**:
  - Automatically cleans up and removes `.m3u8` playlist files.
  - Renames `.cue` and `.log` files to match the album:
    - **1 Disc**: `{Album Artist} - {Album Name}.cue` / `.log`
    - **Multi-Disc**: `{Album Artist} - {Album Name} [Disc 1].cue`, `[Disc 2].cue`, etc.
  - Updates internal `FILE "..."` references inside `.cue` files to match the newly renamed audio files.
- **Supported Formats**: `.flac`, `.mp3`, `.m4a`, `.mp4`, `.ogg`, `.opus`, `.wav`, `.aiff`.

---

## Written Metadata Tags

`mp3tag` standardizes all tags across all supported audio containers. Tag names in Vorbis Comments are strictly **UPPERCASE**.

| Field Name | Vorbis Comment (FLAC / OGG / Opus) | ID3v2.4 Frame (MP3) | MP4 Atom (M4A) | Format & Description | Example |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Title** | `TITLE` | `TIT2` | `©nam` | Track title | `BOOMBAYAH` |
| **Artist** | `ARTIST` | `TPE1` | `©ART` | Track artist | `BLACKPINK` |
| **Album** | `ALBUM` | `TALB` | `©alb` | Album title | `SQUARE ONE` |
| **Album Artist** | `ALBUMARTIST` | `TPE2` | `aART` | Album artist | `BLACKPINK` |
| **Track Number** | `TRACKNUMBER` | `TRCK` | `trkn` | Zero-padded 2-digit number | `01`, `02`, `12` |
| **Track Total** | `TRACKTOTAL` | `TRCK` | `trkn` | Total tracks on current disc | `12` |
| **Disc Number** | `DISCNUMBER` | `TPOS` | `disk` | Unpadded number | `1`, `2` |
| **Disc Total** | `DISCTOTAL` | `TPOS` | `disk` | Total discs in album | `2` |
| **Year** | `YEAR` | `TDRC` / `TYER` | `©day` | 4-digit release year only | `2020` |
| **Catalog Number** | `CATALOGNUMBER` | `TXXX:CATALOGNUMBER` | `----:com.apple.itunes:CATALOGNUMBER` | Per-disc catalog number | `IMAE-00046` |
| **Label / Publisher** | `LABEL` | `TPUB` | `----:com.apple.itunes:LABEL` | Record label (omitted when empty or placeholder like `[no label]`) | `YG Entertainment` |
| **Media Format** | `MEDIA` | `TMED` | `----:com.apple.itunes:MEDIA` | Auto-detected: `WEB` (24-bit FLAC), `CD` (.cue/.log), or empty | `WEB`, `CD` |
| **Event** | `EVENT` | `TXXX:EVENT` | `----:com.apple.itunes:EVENT` | Convention/event code (e.g. `C89`, `M3-50`, `Reitaisai 21`, or empty if none) | `C89`, `M3-50` |
| **Comment** | `COMMENT` | `COMM` | `©cmt` | Album disambiguation / info | `Deluxe Edition` |
| **Front Cover Art** | Picture Block / `METADATA_BLOCK_PICTURE` | `APIC` | `covr` | Front cover (JPEG, only with `--cover`) | `image/jpeg` |

> [!NOTE]
> All other legacy or redundant tags are removed before writing:
> - `DISC` (removed; replaced by `DISCNUMBER`)
> - `TRACK` (removed; replaced by `TRACKNUMBER`)
> - All `musicbrainz_*` tags (completely stripped)
> - All encoder, ripping, and replaygain tags (`encoder`, `tsse`, `replaygain_*`, etc.)

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
4. Display a preview table of aligned tracks, **new filenames**, **per-disc catalog numbers**, **MEDIA**, and **EVENT**.
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

### 4. Cover Art (albumart.jpg & --cover)
- **Automatic Local Standardization**: Whenever `mp3tag` runs, it detects any existing cover image in the folder (e.g. `cover.png`, `folder.jpg`, or a random filename) and standardizes it to `albumart.jpg` (converting to JPEG if needed).
- **Embedding into Audio Tags**: Tag embedding is disabled by default. Pass `--cover` to embed `albumart.jpg` into your audio files (or download from Cover Art Archive if no local cover is found):
```bash
mp3tag some_folder --cover
```

### 5. Custom Search Query
If a folder name is unconventional or shortened, supply a custom query:
```bash
mp3tag some_folder -q "Steins Gate Original Soundtrack"
```

### 6. Direct MusicBrainz Release or Release-Group ID / URL
Directly tag from an exact MusicBrainz Release or Release-Group URL or UUID:
```bash
# Direct Release URL or UUID
mp3tag some_folder --id "https://musicbrainz.org/release/a03a7a82-837c-4e76-af89-15e6daca4caa"
mp3tag some_folder --id "a03a7a82-837c-4e76-af89-15e6daca4caa"

# Direct Release Group URL or UUID
mp3tag some_folder --id "https://musicbrainz.org/release-group/740e262e-9779-4316-adb2-0be948a414a7"
```
You can also paste release or release-group URLs directly into the interactive `[u]RL/ID` prompt or the manual search prompt when prompted.

### 7. Event Tagging (-e, --event)
Override or explicitly specify the convention/event code for the `EVENT` tag:
```bash
mp3tag some_folder -e "C89"
mp3tag some_folder -e "M3-56"
mp3tag some_folder -e "Reitaisai 21"
```
Event names like `"Comic Market 109"` or `"M3-2025 Fall"` are automatically normalized to their canonical short forms (`C109`, `M3-56`). If omitted and no event is detected from folder names, existing tags, or MusicBrainz disambiguation, the tag is left empty.

---

## CLI Options

```text
usage: mp3tag [-h] [-y] [-n] [--cover] [-q QUERY] [-e EVENT] [--id ID] [-v] folder [folder ...]

positional arguments:
  folder             Directory containing the album or audio files

options:
  -h, --help         show this help message and exit
  -y, --yes          Non-interactive mode (auto-select top match & apply)
  -n, --dry-run      Preview tags and changes without writing to disk
  --cover            Download and embed front cover art (and save albumart.jpg). Default: disabled
  -q, --query QUERY  Override search query instead of using the folder name
  -e, --event EVENT  Override or specify the EVENT tag (e.g. C89, M3-50, Reitaisai 21)
  --id ID            MusicBrainz Release ID (UUID) or URL to fetch directly
  -v, --verbose      Enable verbose debug output
```
