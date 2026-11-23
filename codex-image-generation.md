---
name: codex-image-generation
description: "need an image, icon or PNG drawn (Claude has no image output): `codex exec` generates them (`image_generation` feature) unsandboxed; its own resize/quantize flattens drafts into blobs, so ask for originals unchanged and downscale with magick; `-i` ref before the prompt swallows it (`No prompt provided via stdin.`); exact shapes (rotated arrowheads) drawn after with `magick -draw`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 36569284-c567-4835-8f97-cf71229c0769
  modified: 2026-11-23T15:08:10.855Z
---

Claude cannot produce raster images; Codex can. `codex features list`
shows `image_generation` (stable, enabled; verified against codex-cli
0.163.0).

- Run `codex exec --skip-git-repo-check --ephemeral -s workspace-write -C <dir> '<brief>'`
  with the sandbox off (Bash `dangerouslyDisableSandbox`), as for the
  other Codex entry points in [[codex-plugin-bugs]]. One tool call per
  image; about 1–2 minutes each.
- Files land in `~/.codex/generated_images/<session>/exec-*.png`, about
  1254×1254 RGBA with a real transparent background; the brief has Codex
  copy them where you want.
- Left to post-process, Codex quantized to 3–4 colours and thresholded
  alpha, which turned two of four drafts into solid silhouettes. Tell it
  "do not resize or post-process; copy unchanged", then
  `magick in.png -filter Lanczos -resize 512x512 -strip PNG32:out.png`.
- `-i <file>` attaches a reference image and keeps redrawing from it,
  but it takes several values: a prompt placed after it is swallowed and
  Codex prints `No prompt provided via stdin.` Put the prompt before
  `-i`.
- Image generation does not hold exact geometry: asked repeatedly for
  arrowheads rotated along diverging branches, it drew them pointing
  straight ahead. Have Codex draw the picture without that element, then
  draw it with `magick in.png -fill '#hex' -draw 'polygon x,y …'`; the
  built-in MSVG renderer (no rsvg or inkscape on the box) drops SVG
  strokes and misplaces polygons.
- An unsandboxed Bash `cd` moves the session's working directory: start
  such calls with `cd <repo> &&` or use `-C`.
- Files Codex writes are mode 600; `chmod 644` before a human opens them
  over a network share.
