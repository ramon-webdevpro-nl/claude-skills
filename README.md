# claude-skills

Claude Code skills for local image processing — no API calls, no cost.

## Skills

### [gimp-inkscape](skills/gimp-inkscape/SKILL.md)

Full local image processing toolkit using GIMP, Inkscape, ImageMagick, FFmpeg, ExifTool, OptiPNG, jpegoptim, and pdftoppm.

**Install:**
```bash
npx skills add ramon-webdevpro-nl/claude-skills@gimp-inkscape
```

**Covers:**
- Resize, crop, batch operations (ImageMagick)
- Text overlays and watermarks (ImageMagick)
- SVG creation and SVG → PNG export (Inkscape)
- Photo editing and compositing (GIMP)
- GIF creation and frame extraction (FFmpeg)
- Metadata read/strip/write (ExifTool)
- Lossless PNG/JPEG compression (OptiPNG, jpegoptim)
- PDF → image conversion (pdftoppm)
- WebP conversion (ImageMagick)
- Decision guide: which tool wins for each task

**Requires:** Linux/macOS with the relevant tools installed. On Ubuntu: `apt install gimp inkscape imagemagick ffmpeg libimage-exiftool-perl optipng jpegoptim poppler-utils`
