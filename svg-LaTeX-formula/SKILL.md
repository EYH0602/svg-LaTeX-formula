---
name: svg-LaTeX-formula
description: >
  Convert LaTeX math formulas from Draw.io files into clean, portable SVG files.
  Use this skill whenever the user wants to: export LaTeX/math formulas as SVGs,
  convert Draw.io formulas to SVG, get math equations out of Draw.io for use in
  other tools (Excalidraw, Lark, Google Slides, Canva, etc.), or mentions needing
  LaTeX formulas as images/vectors. Also use when the user mentions pdf2svg or
  drawio-to-svg conversion for math content.
---

# LaTeX Formula to SVG Conversion

This skill converts LaTeX math formulas authored in Draw.io into clean, standard SVG files that work in any tool.

## Why This Exists

Draw.io is one of the few free tools that renders LaTeX math, but its SVG export produces files with proprietary XML that other tools can't read. The workaround is to export as PDF first (which produces clean vector output), then convert the PDFs to SVGs using `pdf2svg`.

## Prerequisites

Before starting, verify both tools are available:

```bash
drawio --help 2>/dev/null && echo "drawio: OK" || echo "drawio: NOT FOUND"
pdf2svg 2>/dev/null; [ $? -le 1 ] && echo "pdf2svg: OK" || echo "pdf2svg: NOT FOUND"
```

If `drawio` is missing, the user needs [Draw.io Desktop](https://github.com/jgraph/drawio-desktop/releases).
If `pdf2svg` is missing: `sudo apt install pdf2svg` (Ubuntu/Debian), `brew install pdf2svg` (macOS), or `sudo pacman -S pdf2svg` (Arch).

## The Pipeline

### Step 1: Export Draw.io pages to PDF

Each page (or selected block) in the `.drawio` file becomes one PDF. Use these exact export settings for clean, tight output:

- **Format**: PDF
- **Border Width**: 0
- **Background**: Transparent
- **Crop**: Enabled (fits to content)

```bash
# Export a single page (0-indexed)
drawio -x -f pdf --border 0 --transparent --crop -p <page_index> -o "<output>.pdf" "<input>.drawio"

# Export all pages — loop over page indices
# First, count pages (each <diagram> element is a page):
PAGE_COUNT=$(grep -c '<diagram' "<input>.drawio")
for i in $(seq 0 $((PAGE_COUNT - 1))); do
  drawio -x -f pdf --border 0 --transparent --crop -p "$i" -o "<output_dir>/page-${i}.pdf" "<input>.drawio"
done
```

When the user has multiple `.drawio` files, process each one. Use subagents to parallelize if available — one per file speeds things up significantly.

### Step 2: Convert PDFs to SVG

```bash
# Single file
pdf2svg "<input>.pdf" "<output>.svg"

# Batch — convert all PDFs in a directory
for pdf in "<pdf_dir>"/*.pdf; do
  svg="<svg_dir>/$(basename "${pdf%.pdf}.svg")"
  pdf2svg "$pdf" "$svg"
done
```

### Step 3: Verify output

After conversion, do a quick sanity check:
- Confirm each SVG file is non-empty and valid XML
- Optionally report file sizes so the user knows what they're working with

```bash
for svg in "<svg_dir>"/*.svg; do
  size=$(wc -c < "$svg")
  echo "$(basename "$svg"): ${size} bytes"
done
```

## Directory Convention

When the user doesn't specify output locations, suggest this layout relative to the `.drawio` file:

```
formulas/          ← .drawio source files
formulas-pdf/      ← intermediate PDFs (can be deleted after)
svgs/              ← final SVG output
```

## Handling Multiple Formulas

A common pattern is one `.drawio` file with many pages, each containing a single formula. In this case:

1. Count the pages in the file
2. Export each page as a separate PDF (using `-p <index>`)
3. Convert all PDFs to SVGs
4. Name the output files descriptively if the page names are available

If the user has multiple `.drawio` files instead, process each file independently. Parallelize with subagents when possible — each file is fully independent.

## Troubleshooting

**"drawio: command not found"** — Draw.io Desktop must be installed. On Linux, the AppImage or snap may not add `drawio` to PATH automatically. Check `/snap/bin/drawio` or the AppImage location.

**pdf2svg produces empty SVG** — The input PDF may be blank. Check that the Draw.io page actually has content and that the `--crop` flag didn't crop away everything (this can happen if the formula is outside the page bounds).

**SVG looks correct but won't import** — Some tools are picky about SVG features. The pdf2svg output is generally very compatible, but if issues arise, try opening the SVG in Inkscape and re-saving as "Plain SVG".
