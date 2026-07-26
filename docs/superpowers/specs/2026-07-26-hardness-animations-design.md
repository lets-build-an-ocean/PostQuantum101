# Hardness animations — design spec

Date: 2026-07-26

## Goal

Give the README a visual, "playing on load" demonstration of the two hard
problems the repo teaches: RSA's factoring hardness and ML-DSA's lattice
(SVP-style) hardness. Must render inline on github.com without a click —
that rules out interactive 3D and GitHub's auto-play-less video player, and
points at GIF.

## Scope

Two clips, built in this order:
1. RSA factoring hardness
2. ML-DSA lattice hardness

## Visual design

Both clips share one visual language: green = easy forward direction,
red = hard reverse direction. Abstract/mathematical style (numbers, vectors,
lattice points as clean 3D geometry) — not skeuomorphic (no padlocks/gears).

### RSA clip (~8-12s loop)

- Two primes `p`, `q` shown as labeled 3D solids, slide together and
  multiply into a large number `n`. Fast motion, green, "easy" tag.
- Cut to: only `n` is visible (implying an attacker who only has the public
  modulus).
- Many trial-factor guesses tick/scatter past `n`, trying to split it back
  into `p, q`. Slow motion, red, "hard" tag.
- Loop back to start.

### ML-DSA clip (same length/rhythm)

- 3D lattice point grid; basis vectors forming the public matrix `A`.
- Short secret vector `s1` hops from the origin along the lattice to
  endpoint `t = A·s1`. Green, "easy" tag (forward direction).
- Cut to: many long random guess-vectors from the origin try to land on `t`
  without knowing `s1`. None converge. Camera pulls back to show the
  lattice's scale. Red, "hard" tag.
- Loop back to start.

## Pipeline

- Manim Community Edition (ManimCE), one Scene class per clip.
- Source kept in repo under `animations/`:
  - `animations/rsa_scene.py`
  - `animations/ml_dsa_scene.py`
- Render headless to MP4, then convert to a palette-optimized GIF via
  ffmpeg two-pass (palettegen/paletteuse) to avoid banding and keep file
  size down. No transparency (GIF alpha edges are jagged) — fixed dark
  charcoal background (not pure black, so it doesn't look like a hole in
  GitHub's light theme).
- Rendered GIFs committed to `assets/`:
  - `assets/rsa-hardness.gif`
  - `assets/ml-dsa-hardness.gif`

## README integration

Embed each GIF directly under the matching existing section, in both
language files:

- README.en.md — under "Where does RSA's security come from?" and under
  the ML-DSA "easy way to picture it" / hardness section.
- README.md (Persian) — under "راز امنیت RSA کجاست؟" and the matching
  ML-DSA section.

## Verification

- Render locally, eyeball loop smoothness and label readability at README
  embed width.
- Target file size under ~3-5MB per GIF so the README stays fast to load.
- After pushing, spot-check the rendered README on github.com in both
  light and dark theme.

## Environment note

This machine is missing `ffmpeg`, `libcairo2-dev`, `libpango1.0-dev`,
`pkg-config` (system libs ManimCE needs) and the `manim` pip package.
Installing the system libs needs sudo, which the assistant cannot run
non-interactively — the user installs these themselves before
implementation can render anything:

```
sudo apt-get update && sudo apt-get install -y ffmpeg libcairo2-dev libpango1.0-dev pkg-config python3-dev
pip install manim
```

## Out of scope / explicitly not doing

- No interactive three.js/WebGL version (Path B from the earlier
  discussion) — may be a future follow-up, not part of this spec.
- No build tooling/Makefile for re-rendering — two scenes, re-render
  manually with the standard `manim` CLI when the source changes.
