---
name: gimp-inkscape
description: Create and manipulate images using GIMP, Inkscape, ImageMagick, FFmpeg, ExifTool, OptiPNG, jpegoptim, and pdftoppm via CLI. Use for: resizing, cropping, compositing, text overlays, watermarks, color correction, format conversion, WebP export, SVG creation, SVG-to-PNG export, OG images, social banners, logo manipulation, batch processing, lossless compression, metadata read/strip, GIF creation, animation, PDF-to-image. Triggers: gimp, inkscape, imagemagick, resize image, crop image, convert image, svg to png, add text to image, watermark, composite, image editing, banner, social card, compress image, strip metadata, gif, pdf to image, webp.
allowed-tools: Bash(gimp *), Bash(inkscape *), Bash(convert *), Bash(mogrify *), Bash(identify *), Bash(ffmpeg *), Bash(exiftool *), Bash(optipng *), Bash(jpegoptim *), Bash(pdftoppm *)
---

# GIMP + Inkscape + ImageMagick Image Skill

Local image creation and manipulation. No API calls, no cost, no internet required.

## Decision Guide — Pick the Right Tool

| Task | Use | Notes |
|---|---|---|
| Simple resize | **ImageMagick** | 10× faster than GIMP, one liner |
| Batch resize folder | **ImageMagick mogrify** | Purpose-built for batch |
| SVG → PNG | **Inkscape** | Vector quality, exact dimensions |
| Text overlay / watermark | **ImageMagick** | Reliable; GIMP Script-Fu text is flaky |
| Complex typography | **Inkscape SVG** | Pixel-perfect vector text |
| Format conversion | **ImageMagick** | One liner, handles WebP too |
| Compositing / layers | **GIMP** | Full layer control |
| Color correction | **GIMP** | Curves/levels more precise than IM |
| PNG compression | **OptiPNG** | Lossless, 20–50% savings |
| JPEG compression | **jpegoptim** | Lossless recompression |
| Metadata read/strip | **ExifTool** | The industry standard |
| GIF / animation | **FFmpeg** | Nothing else handles this |
| PDF → image | **pdftoppm** | Clean, DPI-controllable |
| Photo editing (advanced) | **GIMP** | Healing, clone stamp, curves |

### Optional installs for two specific tasks
- **Background removal:** `pip install rembg` then `rembg i input.png output.png`
- **AI upscaling:** [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) — significant quality boost over interpolation upscaling

---

## ImageMagick — CLI Reference

### Format conversion
```bash
convert input.png output.jpg
convert input.jpg output.webp
convert input.png -quality 92 output.jpg
```

### Resize
```bash
# Exact dimensions (may distort)
convert input.jpg -resize 1200x630! output.jpg

# Fit within box (preserves aspect ratio)
convert input.jpg -resize 1200x630 output.jpg

# Fill and crop to exact dimensions (centered)
convert input.jpg -resize 1200x630^ -gravity Center -extent 1200x630 output.jpg
```

### Batch resize entire folder (mogrify)
```bash
# Resize all JPEGs in place
mogrify -resize 800x600 /path/to/folder/*.jpg

# Resize and output to a different folder
mogrify -resize 800x600 -path /output/folder /input/folder/*.jpg

# Convert all PNGs to WebP
mogrify -format webp /path/to/folder/*.png
```

### Text overlay / watermark
```bash
# Simple text in bottom-right corner
convert input.jpg \
  -font DejaVu-Sans -pointsize 36 \
  -fill white -annotate +20+40 "© ReshareAI" \
  output.jpg

# Centered text with shadow
convert input.jpg \
  -font DejaVu-Sans-Bold -pointsize 72 \
  -fill black -annotate +202+302 "Headline" \
  -fill white -annotate +200+300 "Headline" \
  output.jpg

# Semi-transparent watermark tiled across image
convert input.jpg \
  -font DejaVu-Sans -pointsize 48 -fill "rgba(255,255,255,0.3)" \
  -gravity Center -annotate 30x30+0+0 "WATERMARK" \
  output.jpg
```

### Composite: place logo/image on top
```bash
# Place logo.png bottom-right with 10px padding
convert base.jpg \
  logo.png \
  -gravity SouthEast -geometry +10+10 \
  -composite output.jpg

# Place with opacity
convert base.jpg \
  \( logo.png -alpha set -channel Alpha -evaluate multiply 0.5 \) \
  -gravity Center -composite output.jpg
```

### Color adjustments
```bash
# Brightness / contrast
convert input.jpg -brightness-contrast 10x20 output.jpg

# Saturation
convert input.jpg -modulate 100,130,100 output.jpg

# Grayscale
convert input.jpg -colorspace Gray output.jpg

# Sharpen
convert input.jpg -unsharp 0x1 output.jpg
```

### Get image info
```bash
identify input.jpg
identify -verbose input.jpg | grep -E "Geometry|Colorspace|Format|filesize"
```

---

## ImageMagick — WebP

ImageMagick has the WebP delegate built in (confirmed on this system):

```bash
# PNG/JPEG → WebP
convert input.png -quality 85 output.webp

# WebP → PNG
convert input.webp output.png

# Batch convert all JPEGs to WebP
mogrify -format webp -quality 85 /path/to/folder/*.jpg
```

---

## FFmpeg — GIF & Animation

### Video → GIF
```bash
# Basic
ffmpeg -i input.mp4 -vf "fps=10,scale=800:-1" output.gif

# High quality (uses palette)
ffmpeg -i input.mp4 -vf "fps=12,scale=800:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -vf "fps=12,scale=800:-1:flags=lanczos,paletteuse" output.gif

# Clip a section (start 5s, duration 3s)
ffmpeg -ss 5 -t 3 -i input.mp4 -vf "fps=10,scale=600:-1" output.gif
```

### Extract frames from video
```bash
# Extract every frame as PNG
ffmpeg -i input.mp4 /output/frame_%04d.png

# Extract 1 frame per second
ffmpeg -i input.mp4 -vf fps=1 /output/frame_%04d.png

# Extract a single frame at timestamp
ffmpeg -ss 00:00:05 -i input.mp4 -frames:v 1 output.png
```

### Image sequence → video or GIF
```bash
# Frames → MP4
ffmpeg -framerate 24 -i frame_%04d.png -c:v libx264 output.mp4

# Frames → GIF
ffmpeg -framerate 10 -i frame_%04d.png output.gif
```

---

## ExifTool — Metadata

### Read metadata
```bash
exiftool input.jpg
exiftool -s input.jpg                          # short tag names
exiftool -ImageSize -DateTimeOriginal input.jpg  # specific fields
```

### Strip all metadata (privacy / pre-publish)
```bash
exiftool -all= input.jpg                       # strips in place (saves backup)
exiftool -all= -overwrite_original input.jpg   # no backup
exiftool -all= -overwrite_original /folder/*.jpg  # batch strip
```

### Copy EXIF from one file to another
```bash
exiftool -TagsFromFile source.jpg target.jpg
```

### Write metadata
```bash
exiftool -Artist="Ramon" -Copyright="© 2026 ReshareAI" input.jpg
exiftool -all= -overwrite_original -Artist="Ramon" /folder/*.jpg
```

---

## OptiPNG — Lossless PNG Compression

```bash
# Optimize (overwrites in place, creates backup .bak)
optipng input.png

# No backup
optipng -o 7 input.png           # -o 7 is max compression (slower)

# Output to new file
optipng -out output.png input.png

# Batch
optipng /path/to/folder/*.png

# Check savings without writing
optipng -simulate input.png
```

Typical savings: 20–50% file size reduction with zero quality loss.

---

## jpegoptim — Lossless JPEG Compression

```bash
# Lossless optimization (in place)
jpegoptim input.jpg

# With target file size
jpegoptim --size=200k input.jpg

# Strip all metadata + optimize
jpegoptim --strip-all input.jpg

# Batch (folder)
jpegoptim /path/to/folder/*.jpg

# Batch with max quality floor
jpegoptim --max=90 /path/to/folder/*.jpg
```

---

## Poppler — PDF → Image

```bash
# All pages → PNG at 150 DPI (output: file-000001.png, file-000002.png ...)
pdftoppm -r 150 -png input.pdf /output/page

# All pages → JPEG
pdftoppm -r 150 -jpeg input.pdf /output/page

# Single page (page 1 only)
pdftoppm -r 300 -png -f 1 -l 1 input.pdf /output/page

# High DPI for print-quality extraction
pdftoppm -r 300 -png input.pdf /output/page
```

---

## Inkscape — CLI Reference

### SVG → PNG export (exact dimensions)
```bash
inkscape input.svg \
  --export-type=png \
  --export-filename=output.png \
  --export-width=1200 \
  --export-height=630
```

### Export specific layer
```bash
inkscape input.svg \
  --export-type=png \
  --export-filename=output.png \
  --export-id=layer1 \
  --export-id-only
```

### SVG → PDF
```bash
inkscape input.svg --export-type=pdf --export-filename=output.pdf
```

### Create SVG from scratch (write file, then export)
Write a `.svg` file using the Write tool, then export with Inkscape. SVG template:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg width="1200" height="630" viewBox="0 0 1200 630"
     xmlns="http://www.w3.org/2000/svg">

  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#4C1D95"/>
      <stop offset="100%" style="stop-color:#1E1B4B"/>
    </linearGradient>
  </defs>

  <rect width="1200" height="630" fill="url(#bg)"/>

  <!-- Text renders perfectly — no hallucination like GIMP/FLUX -->
  <text x="100" y="280"
        font-family="Inter, Arial, sans-serif"
        font-size="96" font-weight="700"
        fill="white">BrandName</text>

  <text x="100" y="360"
        font-family="Inter, Arial, sans-serif"
        font-size="32" fill="rgba(255,255,255,0.7)">Tagline goes here</text>

</svg>
```

---

## GIMP — Script-Fu Batch Reference

Use GIMP for: compositing, color correction, photo editing, alpha channel work. For simple tasks prefer ImageMagick.

### Resize to exact dimensions
```bash
gimp --no-interface --batch '(let* (
  (image (car (gimp-file-load RUN-NONINTERACTIVE "/path/input.png" "input.png")))
  (drawable (car (gimp-image-get-active-drawable image))))
  (gimp-image-scale-full image 1200 630 INTERPOLATION-LINEAR)
  (file-png-save RUN-NONINTERACTIVE image (car (gimp-image-get-active-drawable image))
    "/path/output.png" "output.png" 0 9 1 1 1 1 1))
(gimp-quit 0)'
```

### Scale then crop (fill mode — covers full canvas)
```bash
gimp --no-interface --batch '(let* (
  (image (car (gimp-file-load RUN-NONINTERACTIVE "/path/input.jpg" "input.jpg")))
  (w (car (gimp-image-width image)))
  (h (car (gimp-image-height image)))
  (target-w 1200) (target-h 630)
  (scale (max (/ target-w w) (/ target-h h)))
  (new-w (inexact->exact (round (* w scale))))
  (new-h (inexact->exact (round (* h scale)))))
  (gimp-image-scale-full image new-w new-h INTERPOLATION-LINEAR)
  (gimp-image-crop image target-w target-h
    (/ (- new-w target-w) 2) (/ (- new-h target-h) 2))
  (file-png-save RUN-NONINTERACTIVE image (car (gimp-image-get-active-drawable image))
    "/path/output.png" "output.png" 0 9 1 1 1 1 1))
(gimp-quit 0)'
```

### Add semi-transparent colour overlay
```bash
gimp --no-interface --batch '(let* (
  (image (car (gimp-file-load RUN-NONINTERACTIVE "/path/input.png" "input.png")))
  (overlay (car (gimp-layer-new image
    (car (gimp-image-width image))
    (car (gimp-image-height image))
    RGBA-IMAGE "overlay" 60 LAYER-MODE-NORMAL))))
  (gimp-image-insert-layer image overlay 0 -1)
  (gimp-context-set-foreground (list 0 0 0))
  (gimp-edit-fill overlay FILL-FOREGROUND)
  (file-png-save RUN-NONINTERACTIVE image (car (gimp-image-flatten image))
    "/path/output.png" "output.png" 0 9 1 1 1 1 1))
(gimp-quit 0)'
```

### Composite: place image B on top of image A
```bash
gimp --no-interface --batch '(let* (
  (base (car (gimp-file-load RUN-NONINTERACTIVE "/path/base.png" "base.png")))
  (overlay-img (car (gimp-file-load RUN-NONINTERACTIVE "/path/overlay.png" "overlay.png")))
  (overlay-layer (car (gimp-layer-new-from-drawable
    (car (gimp-image-get-active-drawable overlay-img)) base))))
  (gimp-image-insert-layer base overlay-layer 0 -1)
  (gimp-layer-set-offsets overlay-layer 50 50)
  (file-png-save RUN-NONINTERACTIVE base (car (gimp-image-flatten base))
    "/path/output.png" "output.png" 0 9 1 1 1 1 1))
(gimp-quit 0)'
```

### Brightness / contrast / saturation
```bash
gimp --no-interface --batch '(let* (
  (image (car (gimp-file-load RUN-NONINTERACTIVE "/path/input.jpg" "input.jpg")))
  (drawable (car (gimp-image-get-active-drawable image))))
  (gimp-brightness-contrast drawable 20 30)
  (gimp-drawable-hue-saturation drawable HUE-RANGE-ALL 0 0 25 0)
  (file-jpeg-save RUN-NONINTERACTIVE image drawable
    "/path/output.jpg" "output.jpg" 0.92 0 0 0 "" 0 1 0 2 0))
(gimp-quit 0)'
```

---

## Recommended Workflow: OG Image / Social Banner

1. **Write** an SVG file (exact layout, text, colors) using the Write tool
2. **Export** with Inkscape at target dimensions — text is pixel-perfect
3. **Optimize** with OptiPNG before publishing

```bash
# Step 1: write /tmp/card.svg

# Step 2: export
inkscape /tmp/card.svg \
  --export-type=png \
  --export-filename=/output/og.png \
  --export-width=1200 \
  --export-height=630

# Step 3: optimize
optipng -o 5 /output/og.png
```

---

## Tips

- **Text accuracy:** Inkscape SVG > ImageMagick `-annotate` > GIMP Script-Fu (avoid for text)
- **Speed hierarchy:** ImageMagick > GIMP for simple ops; GIMP wins for complex compositing
- **GIMP `--no-interface`** — always end scripts with `(gimp-quit 0)`
- **Script-Fu paths must be absolute** — no `~/` shortcuts
- **GIMP opacity** is 0–100 (integer), not 0.0–1.0
- **Inkscape 1.1** uses `--export-type` + `--export-filename` (older versions used `--export-png`)
- **ImageMagick `!` suffix** on geometry forces exact dimensions ignoring aspect ratio (`1200x630!`)
