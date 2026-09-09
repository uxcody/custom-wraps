# AGENTS.md

This file provides guidance to AI coding agents working in this repository. `CLAUDE.md` is a symlink to this file.

## What this repository is

An **asset and documentation repository**, not a software project. It distributes PNG wrap templates and example designs that Tesla owners load into their vehicle's Paint Shop (Toybox → Paint Shop → Wraps) via the mobile app or a USB drive.

There is no build system, no test suite, no linter, no CI, and no source code. Changes are edits to PNG assets and Markdown. Verification means checking image invariants and that README links resolve.

## Structure

Every vehicle variant is a self-contained top-level directory with an identical four-part shape:

```
<variant>/
  template.png        # the UV layout the owner paints on (white = paintable area)
  vehicle_image.png   # thumbnail used by the root README table
  README.md           # per-variant page: template link + example gallery
  example/*.png       # finished example wraps for this variant
```

`images/` is the only exception: it holds screenshots referenced by the root README, not vehicle assets.

The variant directories are the unit of change. Adding a vehicle means adding one directory in this exact shape **and** adding a cell to the table in the root `README.md` — the root table is hand-maintained and is the only index, so a new directory is invisible until it is listed there.

### Templates are per-variant, never shared

All twelve `template.png` files are distinct, and same-named examples differ across variants (`model3/example/Camo.png` and `modely/example/Camo.png` are different files). Each is a UV unwrap of that specific body, so an asset is never copied between directories — a design must be re-rendered against the target variant's template.

Variants are split finely by trim where the body geometry differs: Model 3 (2024+) has separate base and performance directories, and Model Y (2025+) has base, premium, and performance. Directory naming is `<model>[-<year>][-<trim>]`, all lowercase.

### Per-variant README is a template

Every `<variant>/README.md` is the same 38-line document with only the H1 changed (Cybertruck's is longer only because it has more examples). When adding one, copy an existing file and change the title and the example list. The structure is fixed: intro linking to the main page, `## Template` with a clickable `template.png?raw=true` preview at width 600, `## Examples` as a `<p>` of `<a>`/`<img>` pairs at width 150 sorted alphabetically by filename, then a back-link footer. Cross-repo links point at `https://github.com/teslamotors/custom-wraps`, not relative paths.

## Asset invariants

These come from the vehicle's own loader limits documented in the root README, and every committed asset satisfies them. Check them after any image change.

- **Format**: PNG, 8-bit RGBA (color type 6). The lone grayscale file is `models-2025-plaid/example/Alpha_Mask.png`, a deliberate mask reference.
- **Dimensions**: 512x512 to 1024x1024. Everything here is 1024x1024 except the Cybertruck, whose template and its 1024x768 examples use the truck's own aspect ratio — do not resize Cybertruck assets to square.
- **Size**: under 1 MB per file. Current worst case is about 910 KB, so headroom is thin; re-check after re-exporting.
- **Filenames**: alphanumerics, underscores, dashes, and spaces only, max 30 characters. Existing examples use `Title_Case_With_Underscores.png`.

Note `vehicle_image.png` (900x1000, or 902x1000 for S/X) is a README thumbnail and is exempt from the loader limits.

## Verifying changes

Read dimensions, color type, and size without extra tooling:

```bash
python3 -c "import struct,sys; d=open(sys.argv[1],'rb').read(26); print(struct.unpack('>IIBB', d[16:26]))" path/to.png
find . -name '*.png' -size +1M
```

Confirm every image referenced by a README exists, since broken links are the common regression when renaming or adding examples:

```bash
python3 -c "
import re, os, glob
bad = [(r, m) for r in glob.glob('*/README.md') + ['README.md']
       for m in re.findall(r'(?:src|href)=\"([^\":]+)\"', open(r).read())
       if not os.path.exists(os.path.join(os.path.dirname(r), m.split('?')[0]))]
print(bad or 'all refs OK')"
```
