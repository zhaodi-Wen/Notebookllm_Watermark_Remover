# NotebookLM Watermark Remover

**[中文文档](README_CN.md)**

> A watermark removal tool built specifically for **NotebookLM** exported PDFs and images. Automatically detects and clears the `🎙 NotebookLM` brand watermark in the bottom-right corner of every page. Supports batch processing, page-by-page preview, and exports as clean PDF, per-page images, or a stitched long image.

## Demo

### Before & After

| Original (with watermark) | Processed (watermark removed) |
|:-:|:-:|
| ![Original](image/origin.png) | ![Processed](image/output.png) |

> Example: NotebookLM exported image-based PDF. The `🎙 NotebookLM` badge in the bottom-right is fully removed without affecting any content.

### Web UI

![Web UI](image/web_ui.png)

Branded interface designed for NotebookLM users:
- **Hero section** — Highlights NotebookLM brand removal with a visual before/after strip
- **3-step workflow** — Upload → Remove → Preview & Export
- **Real-time stats** — Pages processed, text watermarks and image watermarks removed

### Export Per-Page Images

![Export Images](image/output_image.png)

Export each PDF page as a standalone image (PNG/JPG), packaged as a ZIP. DPI is configurable.

---

## Features

- **Pixel-level watermark removal** — For image-based PDFs (NotebookLM, Google Slides exports, etc.) where watermarks are burned into pixels
- **Structural watermark removal** — For traditional PDFs with text/image objects, using keyword scoring to locate watermarks
- **Image file support** — Remove watermarks from standalone PNG/JPG/WEBP images directly
- **Web UI** — Drag-and-drop upload, adjustable sensitivity, per-page preview. All processing is local — no server upload
- **Export PDF** — Download the cleaned PDF
- **Export images** — Export each page as an image (PNG/JPG), configurable DPI, ZIP download
- **Export long image** — Stitch all pages vertically into a single long image
- **Multi-language UI** — 8 languages supported, instant switch via dropdown

## Quick Start

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Start the server

```bash
python app.py
```

Visit http://127.0.0.1:5000

### 3. Usage

1. Drag and drop or click to upload a NotebookLM PDF or image (max 100MB)
2. Choose detection sensitivity (Low / Medium / High) — Medium is recommended
3. Click "Remove Watermark"
4. Preview results page by page
5. Export as PDF, per-page images, or a long image

## Project Structure

```
├── app.py                 # Flask backend + WatermarkRemover engine
├── templates/
│   └── index.html         # Web frontend (branded UI + i18n)
├── requirements.txt       # Python dependencies
├── image/                 # README screenshots
├── uploads/               # Upload directory (auto-created, named by filename)
└── outputs/               # Output directory (auto-created, named by filename)
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | Python + Flask |
| PDF Engine | PyMuPDF (fitz) |
| Image Processing | Pillow + NumPy |
| Frontend | Vanilla HTML/CSS/JS, dark theme, built-in i18n |

## Algorithm

### Image-based PDF (NotebookLM export)

1. **Type detection** — No text + has images in first 3 pages → image-based PDF
2. **Per-page scan** — Bottom-right 30px × 250px region
3. **Smart background sampling** — Find the 5 most "empty" rows, take median color as background
4. **Foreground coverage** — Low threshold (diff > 8) covers all non-background pixels
5. **Smooth transition** — Dilate 3px + Gaussian blur 1px for seamless edge blending
6. **Write back** — `page.replace_image()` replaces the original image data

### Structural PDF

1. **Brand keyword match** — NotebookLM and similar brands trigger directly
2. **Scoring** — Keywords, color, font size multi-factor scoring
3. **Redaction** — PDF redaction annotation permanently removes watermark objects

## Sensitivity Levels

| Level | Description |
|-------|-------------|
| Low | Only removes obvious watermarks, lowest risk of false positives |
| Medium | Recommended — balanced effect and safety |
| High | Aggressive removal, may affect light-colored content |

## Language Support

Instant language switching via the top-right dropdown:

| Language | Code |
|----------|------|
| 🇨🇳 Chinese (Simplified) | zh |
| 🇺🇸 English | en |
| 🇯🇵 日本語 | ja |
| 🇰🇷 한국어 | ko |
| 🇪🇸 Español | es |
| 🇫🇷 Français | fr |
| 🇩🇪 Deutsch | de |
| 🇧🇷 Português | pt |

## API Reference

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/upload` | Upload PDF or image file |
| POST | `/api/process` | Process watermark removal (params: task_id, filename, sensitivity) |
| GET | `/api/preview/{task_id}/{page_num}` | Preview a specific page |
| GET | `/api/page_count/{task_id}` | Get total page count |
| GET | `/api/export/pdf/{task_id}` | Download cleaned PDF |
| GET | `/api/export/images/{task_id}` | Download per-page images ZIP (params: dpi, format) |
| GET | `/api/export/longimage/{task_id}` | Download long image (params: dpi, format) |
| GET | `/api/export/image/{task_id}` | Download cleaned image file |

## Changelog

### v1.4.0
- **New** Image file support: remove watermarks from standalone PNG/JPG/WEBP/BMP files
- **New** Auto file-type detection: PDF and image files follow separate processing paths
- **New** `/api/export/image` endpoint for downloading cleaned image files

### v1.3.0
- **Redesign** Full brand-focused UI for NotebookLM users
  - Hero section with "NotebookLM Watermark Remover" title and feature chips
  - Watermark demo strip showing `🎙 NotebookLM → ✓ Removed`
  - Upload area badge: "✓ NotebookLM PDF Supported"

### v1.2.0
- **New** Long image export: stitch all pages into one vertical image
- **New** 8-language i18n support with instant dropdown switching
- **Improved** Upload/output folders named after original filename

### v1.1.0
- **Fix** Incomplete watermark removal: improved detection threshold and background sampling
- **Fix** Over-coverage damaging content: smart background sampling targets only watermark rows

### v1.0.0
- Initial release: pixel-level + structural watermark removal + Web UI

## Dependencies

```
flask==3.1.0
PyMuPDF==1.25.3
Pillow==11.1.0
gunicorn==23.0.0
```

## License

MIT
