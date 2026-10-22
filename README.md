# mp3tag CLI - Picard & MusicBrainz Automated Tagger

A CLI tool that downloads accurate metadata from the **MusicBrainz (Picard)** database and **Cover Art Archive**, matches tracks intelligently based on folder name, and writes extended tags & cover art directly to music files.

Compatible with **Puddletag**, **Mp3tag**, **MusicBrainz Picard**, and music servers/players like **Feishin**, **Navidrome**, and **Foobar2000**.

---

## Features

- **Folder-based Search**: Cleans folder names (stripping `[FLAC]`, `[320k]`, `(2023)`, etc.), parses `Artist - Album` or `Label - Album`, and finds the release on MusicBrainz.
- **Smart Candidate Scoring**: Ranks candidate releases by matching track counts, media/disc counts, title similarity, artist/label similarity, and release year.
- **High-Resolution Cover Art**: Automatically downloads front cover art from the Cover Art Archive (and release groups) and embeds it directly into audio files and saves `cover.jpg`.
- **Multi-Disc Support**: Handles flat multi-disc filenames (`10.05. Artist - Title.flac`, `1.01. Song.flac`, `01-02 Song.mp3`) as well as subfolders (`Disc 1/`, `CD 2/`).
- **Comprehensive Tags Written**:
  - `album`, `albumartist`, `artist`, `title`
  - `track` / `tracknumber`, `tracktotal` / `totaltracks`
  - `discnumber`, `disctotal` / `totaldiscs`
  - `date`, `year`
  - `catalognumber` / `CATALOGNUMBER`
  - `label` / `LABEL` / `organization`
  - `media` (e.g. CD, Digital Media, Vinyl)
  - `musicbrainz_albumid`, `musicbrainz_trackid`, `musicbrainz_recordingid`, `musicbrainz_artistid`, `musicbrainz_albumartistid`, `musicbrainz_releasegroupid`
  - Front Cover picture embedded into file tags (FLAC Picture, ID3 APIC, MP4 covr, Vorbis block).
- **Supported Formats**: `.flac`, `.mp3`, `.m4a`, `.mp4`, `.ogg`, `.opus`, `.wav`, `.aiff`.

---

## Installation & Setup

`mp3tag` is installed to `~/.local/bin/mp3tag` and added to your `PATH` in `~/.zshrc` and `~/.bashrc`.

You can verify it by running:
```bash
which mp3tag
# Output: /home/azuki/.local/bin/mp3tag
```

---

## Usage

### 1. Basic Usage (Interactive)
Run `mp3tag` with the path to any album folder:
```bash
mp3tag some_folder
```
Example:
```bash
mp3tag "/mnt/tower/data/media/music/5pb. Records - Steins;Gate Original Soundtrack"
```

The tool will:
1. Scan audio files and detect track/disc counts.
2. Search MusicBrainz for matching releases.
3. Show ranked candidate releases with scores.
4. Download cover art from Cover Art Archive.
5. Display a clean preview table of aligned tracks.
6. Ask for confirmation before writing tags to disk.

### 2. Auto-Select Top Match (Non-Interactive / Fast Mode)
Use `-y` or `--yes` to automatically select the top-scoring match and write tags without prompting:
```bash
mp3tag some_folder -y
```

### 3. Dry-Run (Preview Only)
Simulate matching and preview all tag changes without modifying any files:
```bash
mp3tag some_folder -n
```

### 4. Custom Search Query
If a folder name is unconventional or shortened, supply a custom search query:
```bash
mp3tag some_folder -q "SOUND VOLTEX Dimension Gates Phase 1"
```

### 5. Direct MusicBrainz Release ID or URL
Directly tag from an exact MusicBrainz URL or UUID:
```bash
mp3tag some_folder --id "2c7418c5-763f-43ca-ae8c-862ae9fe1393"
```

### 6. Skip Cover Art
```bash
mp3tag some_folder --no-cover
```

---

## CLI Options

```text
usage: mp3tag [-h] [-y] [-n] [-q QUERY] [--id ID] [--no-cover] [-v] folder [folder ...]

positional arguments:
  folder             Directory containing the album or audio files

options:
  -h, --help         show this help message and exit
  -y, --yes          Non-interactive mode (auto-select top match & apply)
  -n, --dry-run      Preview tags and changes without writing to disk
  -q, --query QUERY  Override search query instead of using the folder name
  --id ID            MusicBrainz Release ID (UUID) or URL to fetch directly
  --no-cover         Skip downloading and embedding cover art
  -v, --verbose      Enable verbose debug output
```
