---
openclaw:
  emoji: 📸
  requires:
    bins:
      - node
      - python3
    config:
      - clawimig_dir
  install: |
    cd "$clawimig_dir/scripts" && npm install && pip install -r requirements.txt
---

# clawimig — Instagram Image Pipeline

Process a directory of images for Instagram using sharp (resize/crop/pad) and pilgram (Instagram filters). Reads a `config.json` and writes processed images to an output directory.

## When to Use

Use this skill when the user wants to:
- Prepare photos for Instagram (resize, aspect ratio, padding)
- Apply Instagram-style filters to images (clarendon, gingham, etc.)
- Batch process a folder of images for social media

## Usage Examples

```
"Process my photos for Instagram"
"Run the image pipeline with my config"
"Apply the clarendon filter and resize to portrait format"
"Prepare images in ./raw for Instagram, output to ./ready"
```

## Implementation Details

The pipeline runs in two stages:

**Stage 1 — Geometry (resize.js via Node/sharp):**
- Resize and pad/crop each image to the target Instagram format
- Formats: square (1080×1080), portrait (1080×1350), landscape (1080×566), or custom
- Fit modes: `contain` (pad with background color), `cover` (crop to fill), `fill` (stretch)

**Stage 2 — Filter (process.py via Python/pilgram):**
- Apply one Instagram-style filter from pilgram's 26 built-in options
- Available filters: `_1977`, `aden`, `brannan`, `brooklyn`, `clarendon`, `earlybird`, `gingham`, `hudson`, `inkwell`, `kelvin`, `lark`, `lofi`, `maven`, `mayfair`, `moon`, `nashville`, `perpetua`, `reyes`, `rise`, `slumber`, `stinson`, `toaster`, `valencia`, `walden`, `willow`, `xpro2`
- Set `"filter": null` to skip filtering

**To run the pipeline:**

```bash
cd <project-dir>
python "$clawimig_dir/scripts/process.py" --config config.json
```

**Config file (`config.json`) options:**

| Key | Values | Default | Description |
|-----|--------|---------|-------------|
| `format` | `square`, `portrait`, `landscape`, `custom` | `portrait` | Instagram aspect ratio |
| `custom_dimensions` | `{"width": N, "height": N}` | `null` | Only when `format` is `custom` |
| `background_color` | hex or CSS color | `#ffffff` | Padding color for `contain` fit |
| `fit` | `contain`, `cover`, `fill` | `contain` | How to fill the target canvas |
| `filter` | pilgram filter name or `null` | `null` | Instagram-style color filter |
| `output_format` | `jpeg`, `png`, `webp` | `jpeg` | Output file format |
| `quality` | 1–100 | `90` | Compression quality (JPEG/WebP) |
| `input_dir` | path | `./input` | Folder with source images |
| `output_dir` | path | `./output` | Folder for processed images |

**Supported input formats:** `.jpg`, `.jpeg`, `.png`, `.webp`

## Setup

1. Copy `config.example.json` (from `$clawimig_dir`) to your project and rename to `config.json`
2. Edit `config.json` to match your target format and filter
3. Place images in `input_dir`
4. Run the pipeline command above

## Error Handling

- Missing `input_dir` → exits with a clear error before processing
- Invalid config values → all validation errors printed before any images are touched
- Per-image failures → reported individually; other images continue processing
- Exit code is non-zero if any image failed
