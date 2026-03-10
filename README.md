# svg-LaTeX-formula

A Claude Code skill for converting LaTeX formulas into clean SVG files, ready to embed in any drawing tool.

## The Problem

Many diagramming tools — Lark Docs, Excalidraw, Google Slides, Canva, and others — don't support LaTeX-style math formulas natively. When you need to include equations like `\frac{\partial f}{\partial x}` or `\sum_{i=1}^{n} x_i` in your figures, you're stuck.

**Draw.io** is one of the few free tools that *does* render LaTeX math. So a natural idea is: write your formulas in Draw.io, export them as SVGs, and import the SVGs into whatever tool you're actually using.

But there's a catch — **Draw.io's SVG export doesn't produce clean SVGs**. The exported files contain proprietary XML tags and embedded metadata that other tools can't parse. Try importing a Draw.io SVG into Excalidraw or Lark, and you'll get broken rendering or nothing at all.

## The Solution

The workaround is a two-step pipeline:

1. **Draw.io → PDF**: Export each formula from Draw.io as a cropped, transparent-background PDF (with 0 border width). This produces a clean vector rendering of the LaTeX math.
2. **PDF → SVG**: Use [`pdf2svg`](https://github.com/dawbarton/pdf2svg) to convert each PDF into a standard SVG file.

The resulting SVGs are clean, standard, and import correctly into virtually any tool that supports SVG.

## Prerequisites

- [Draw.io Desktop](https://github.com/jgraph/drawio-desktop/releases) (provides the `drawio` CLI)
- [`pdf2svg`](https://github.com/dawbarton/pdf2svg) — install via your package manager:
  - Ubuntu/Debian: `sudo apt install pdf2svg`
  - macOS: `brew install pdf2svg`
  - Arch: `sudo pacman -S pdf2svg`

## Workflow

1. Create a `.drawio` file with one LaTeX formula per page/block. Use Draw.io's built-in LaTeX support (Insert → Mathematical Typesetting, or wrap text in `$$...$$`).
2. Use the skill (or run manually) to export all pages as individual PDFs, then convert each to SVG.
3. Import the SVGs into your target tool.

## Using as a Claude Code Skill

Install the `svg-LaTeX-formula` directory as a Claude Code skill. Then simply ask Claude to convert your Draw.io formulas to SVGs — it will handle the export and conversion pipeline automatically.

## License

MIT
