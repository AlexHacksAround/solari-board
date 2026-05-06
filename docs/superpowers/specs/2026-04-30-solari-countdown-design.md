# Solari Countdown — Design Spec
_2026-04-30_

## Overview

A single self-contained `countdown.html` in the same project as the Solari display. Shows a `HH:MM:SS` split-flap countdown timer driven by URL parameters. Reuses the same tile CSS, flip animation, and Web Audio click sound as `index.html`.

## Layout

- One row of 8 tiles: `HH : MM : SS`
  - 6 digit tiles (48×64px, animated, split-flap)
  - 2 colon tiles (24×64px, static, no animation, no split line)
- Board is wrapped in a scale container: `transform: scale(factor)` with `transform-origin: top center`
- Scale factor = `(viewport width × width%) / natural board width`
- Recalculated on `window.resize`
- Board is centered on the page

## URL Parameters

| Parameter    | Description                              | Default   |
|--------------|------------------------------------------|-----------|
| `hh`         | Starting hours                           | `0`       |
| `mm`         | Starting minutes                         | `0`       |
| `ss`         | Starting seconds                         | `0`       |
| `width`      | Board width as % of viewport             | `75`      |
| `background` | Page background hex color                | `#888888` |
| `charbg`     | Tile background hex color                | `#000000` |
| `char`       | Character color hex                      | `#FFFFFF` |

## Countdown Logic

- Total seconds = `hh×3600 + mm×60 + ss` computed on load
- `setInterval` fires every 1000ms, decrements total seconds by 1
- Each tick: compute new `HH`, `MM`, `SS` values, compare each digit to previous — only flip digits that changed
- Flip animation: same mechanic as `index.html` (top snaps, bottom rotates out/in)
- Tick sound plays on every digit flip (same Web Audio noise burst as `index.html`)

## Finish Behavior (00:00:00)

- Interval stops
- All digit tiles flash: toggle visibility every 500ms via CSS class
- Finish chime plays: two descending sine tones (880Hz → 440Hz, ~300ms each) via Web Audio
- Flashing continues until user clicks anywhere — then stops, tiles show `00:00:00` static

## Sound

- **Tick**: 12ms high-pass filtered noise burst (same as `index.html`)
- **Chime**: two descending sine tones at zero
- Both controlled by the same mute toggle button (Bootstrap Icons, muted by default, bottom-right)

## Implementation

- Single `countdown.html`, no build step, no shared files with `index.html`
- All tile CSS copied/adapted from `index.html`
- CSS 3D transforms for flip animation
- Web Audio API for tick and chime (no audio files)
- Bootstrap Icons via CDN for mute button
- Inter Bold via Google Fonts

## Edge Cases

- All params missing → starts at `00:00:00` (already at zero, shows static, no countdown)
- Invalid values → fall back to `0`
- `width` out of range → clamp to 10–100%
- `hh`/`mm`/`ss` negative or non-integer → treat as `0`
