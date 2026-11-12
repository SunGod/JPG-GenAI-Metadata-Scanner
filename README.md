<img src="images/a1111_jpg_metadata_scanner_banner.jpg" alt="A1111 JPG Metadata Scanner" width="100%">

# JPG GenAI Metadata Scanner

**Version 1.01** · **macOS** · **Python 3.9+** · **Standard library only**

`jpg_genai_metadata_scanner.py` scans JPEG files for embedded generative-AI metadata, identifies **Draw Things** metadata where there is application-specific evidence, and reports other recognizable AI generation metadata under **GenAI**. It can also distinguish ordinary descriptive metadata from JPEGs with no meaningful embedded metadata.

It is a **metadata detector and inspector**, not a metadata converter. It does not create ComfyUI workflows, convert JPEGs to PNGs, regenerate images, or change metadata inside images. The only optional writes are **CSV reports** and **moving files into classification directories** when you explicitly request those actions.

## What's new in 1.01

Compared with the previous official release (1.00), the scanner:

- Expands beyond A1111-focused detection to recognize generative-AI metadata in EXIF, XMP, IPTC, JPEG comments and supported JPEG application segments.
- Identifies **Draw Things** from its EXIF Software value or distinctive Draw Things generation JSON.
- Groups recognizable A1111-style settings and other GenAI information under **GenAI**, without assigning unsupported A1111 provenance.
- Uses the five **Metadata Type** values below, with an expanded CSV report containing the extracted metadata and detection evidence.
- Adds `--show-genai-only` and independent file organization options for Draw Things and GenAI matches.

## Metadata Types

| Metadata Type | Meaning | Optional move destination |
| --- | --- | --- |
| **Draw Things** | EXIF Software identifies Draw Things, or distinctive Draw Things configuration JSON is detected. | `Draw_Things/` |
| **GenAI** | Recognizable generative-AI generation or processing metadata; not specifically identified as Draw Things. Includes conventional A1111-style settings. | `GenAI/` |
| **Non-GenAI** | Embedded descriptive metadata exists, but no recognized GenAI information is found. | — |
| **None** | No meaningful descriptive metadata is found. Basic JPEG structure (e.g., JFIF headers) does not count. | — |
| **Error** | The JPEG or its metadata cannot be reliably inspected. | — |

The scanner reads **all supported metadata fields before classifying a file**. Specific Draw Things identification takes priority over generic generation-parameter patterns. Each JPEG receives one Metadata Type.

## Requirements and installation

- macOS with **Python 3.9 or newer** accessible as `python3`.
- No third-party packages, network connection, ComfyUI installation, or Draw Things installation required.
- Read access to JPG/JPEG files. CSV reporting requires permission to write the report; file organization requires permission to create directories and move files.

Copy `jpg_genai_metadata_scanner.py` into a convenient location. Run it from Terminal either in the folder being scanned, or provide a directory or JPEG path explicitly.

## Quick start (macOS)

Scan the current folder **without modifying any JPEGs**:

```bash
python3 jpg_genai_metadata_scanner.py .
```

Show only individual **Draw Things** and **GenAI** results, while preserving all five types in the summary and CSV:

```bash
python3 jpg_genai_metadata_scanner.py . --show-genai-only --report metadata.csv
```

Scan subfolders too:

```bash
python3 jpg_genai_metadata_scanner.py . --recursive --show-genai-only --report metadata.csv
```

Organize matched files into `Draw_Things/` and `GenAI/` (these options **move files**):

```bash
python3 jpg_genai_metadata_scanner.py . \
  --show-genai-only \
  --move-draw-things \
  --move-genai \
  --report metadata.csv
```

Add `--recursive` to organize matches found in subdirectories. To inspect *all readable metadata* in one file:

```bash
python3 jpg_genai_metadata_scanner.py --inspect "image10739.jpg"
```

Check the installed script version and command syntax:

```bash
python3 jpg_genai_metadata_scanner.py --version
python3 jpg_genai_metadata_scanner.py --help
```

The final `.` in the examples means **the current directory**. The default scan is nonrecursive. Extension matching is case-insensitive for both `.jpg` and `.jpeg`.

## Command-line reference

| Argument | Action |
| --- | --- |
| `directory` | Optional JPEG file or directory to scan; defaults to `.`. |
| `--inspect FILE.jpg` | Print all readable descriptive and technical JPEG metadata for one file. Cannot be combined with other scan, CSV, filtering or move options. |
| `--recursive` | Include subdirectories during a directory scan. |
| `--report FILE.csv` | Write a CSV with **every scanned file**, including Non-GenAI, None and Error, and extracted readable metadata. |
| `--show-genai-only` | Show **only Draw Things and GenAI individual file lines** on the terminal. All five types remain in CSV and summary totals. |
| `--move-draw-things` | Move **Draw Things** images under `Draw_Things/`. |
| `--move-genai` | Move **GenAI** images under `GenAI/`. |
| `--quiet` | Suppress all individual file lines; still print summary totals. |
| `--version` | Print `JPG GenAI Metadata Scanner 1.01`. |
| `--help`, `-h` | Print command-line help. |

### File-moving behavior and safeguards

- Both move options are **off by default** and can be selected independently or together.
- The folders are created in the scan root (`Draw_Things/`, `GenAI/`), or next to the selected file when scanning one JPG.
- For recursive scans, the relative subdirectory structure is maintained beneath the destination folder (e.g. `events/a.jpg` → `GenAI/events/a.jpg`).
- Existing destination files are **skipped**, not deliberately overwritten. A collision is reported as `SKIPPED_EXISTS` and causes a nonzero exit code.
- The `Draw_Things/` and `GenAI/` destination trees are excluded from **recursive scans when at least one move option is enabled**; if no move option is enabled, a recursive scan can include them.
- JPEG bytes and embedded metadata are not rewritten. Moving a file changes its location on disk; filesystem bookkeeping associated with a move may change.
- Source symbolic-link JPEGs are not moved. Destination directories that are symbolic links are rejected for moves.

The scanner does **not** automatically organize or reclassify files placed in older directory names. Manage existing collections before running bulk moves.

## Terminal output

During a normal scan, the program prints **one tab-separated line per JPEG**:

```text
[Draw Things] image10739.jpg	Source: EXIF Software	Application: Draw Things
[GenAI] image10740.jpg	Source: EXIF UserComment	Application: Unknown
[Non-GenAI] image10741.jpg	Source: EXIF DateTime	Application: Unknown
[None] image10742.jpg	Source: None	Application: Unknown
```

A final summary lists totals for **Draw Things, GenAI, Non-GenAI, None, and Error**, plus move totals if moves were requested. The `--show-genai-only` option suppresses individual Non-GenAI, None, **and Error** lines, but **does not suppress errors in the summary or report**. The separate `--inspect` command intentionally prints multiple lines of metadata for one file.

## CSV report

`--report FILE.csv` writes UTF-8 CSV (with BOM for spreadsheet compatibility), one row for each scanned JPEG. Fields are:

| Column | Contents |
| --- | --- |
| `file` | Source path at the time of scanning (before any requested move). |
| `metadata_type` | Draw Things / GenAI / Non-GenAI / None / Error. |
| `detected_application` | Application identified by supported metadata, if any, or `Unknown`. |
| `detection_source` | Metadata field providing the primary detection evidence. |
| `metadata_sources` | Locations containing readable metadata, such as EXIF, XMP or IPTC. |
| `classification_reason` | Why the type was assigned. |
| `metadata` | JSON array embedded in the CSV cell, with `field`, `source` and `value` for each readable metadata entry. |
| `move_status` | `MOVED`, `SKIPPED_EXISTS`, `FAILED`, `NOT_REQUESTED`, or `NOT_APPLICABLE`. |
| `destination_path` | Move destination when a move was attempted. |

The `metadata` column intentionally includes **ordinary EXIF/IPTC fields as well as GenAI content**. It may be large, especially when Draw Things records lengthy performance-profile arrays. Binary entries are summarized rather than copied in full. A CSV report can therefore contain personal/location metadata already present in the original images; review before sharing it.

## Detection and limitations

- Reads supported JPEG EXIF (including `ImageDescription`, `Software` and `UserComment`), XMP/Extended XMP, IPTC in Photoshop APP13, JPEG COM, and other recognizable APP metadata. The scanner does not exhaustively interpret every proprietary JPEG container or extension.
- **Draw Things** identification accepts either the `Software` tag identifying Draw Things or a distinctive JSON configuration including characteristic `v2` settings. Ordinary A1111-like `Steps`, `Sampler` and `Seed` text in a Draw Things image does **not** override Draw Things identification.
- **GenAI** includes conventional A1111-style infotext and other recognized generation/workflow field patterns; it is a metadata classification, **not** an A1111 application attribution.
- A file with only ordinary EXIF/IPTC tags will typically be **Non-GenAI**. JPEG structural markers alone are **None**.
- Unusual, malformed, encoded or proprietary metadata may be missed or returned as Error; a type of None/Non-GenAI does not certify that no unsupported data exists.
- Reading JPEG headers does not verify or decode the compressed image pixels. The script does not identify visual AI content using image analysis.
- No PNG, WebP, HEIC, MP4, or other format is supported. No image conversion or ComfyUI metadata writing occurs.
- Non-GenAI metadata is preserved in the CSV rather than filtered down to GenAI-only fields.

## Troubleshooting

**`python3: command not found`** — Install a supported Python 3 interpreter and confirm `python3 --version` works.

**`Permission denied`** — Confirm the scan directory is readable; for `--report` or move flags, confirm the destination is writable.

**Unexpected empty results** — The default scan covers only the current directory and `.jpg`/`.jpeg` files. Add `--recursive` to include subfolders, or inspect one specific file with `--inspect`.

**Files were not moved** — Move options must be explicitly supplied. Check the summary and CSV `move_status` for destination collisions (`SKIPPED_EXISTS`), symlinks, or move errors (`FAILED`).

**CSV too large** — Each row may include a full readable EXIF/IPTC inventory, XMP text, and generation JSON. This is expected behavior in 1.01; use the classification and application columns to filter your spreadsheet views.

**Exit status:** `0` means no detected scan/move problems; `1` means at least one JPEG inspection error or failed/skipped move; `2` is used for invalid inspection requests or certain I/O/argument errors.

## Safety

Begin with a **scan-only** run and review its CSV before organizing valuable images. Files are read-only unless you explicitly request a move; the scanner does not create backups or offer an undo operation for moves. Keep a separate backup of important collections.
