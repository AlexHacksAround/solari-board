# Solari Display — Design Spec
_2026-04-29 (updated 2026-04-30)_

## Overview

A single self-contained `index.html` that renders an animated split-flap (Solari) display in the browser. No build tools, no dependencies beyond Inter Bold via Google Fonts and Bootstrap Icons via CDN.

## Layout

- Configurable rows and columns via URL parameters (default: 2 rows × 20 columns).
- Tile size: **48×64px**, Inter Bold 28px.
- A thin 2px gap between top and bottom halves of each tile gives the authentic mechanical look.
- Board is centered on the page.

## URL Parameters

| Parameter      | Description                                      | Default   |
|----------------|--------------------------------------------------|-----------|
| `line1…lineN`  | Text for each row (truncated/padded to `cols`)   | blank     |
| `rows`         | Number of rows                                   | `2`       |
| `cols`         | Number of character columns per row              | `20`      |
| `speed`        | Animation speed multiplier (e.g. `2` = 2× faster) | `1`     |
| `background`   | Page background hex color                        | `#2D327D` |
| `charbg`       | Tile background hex color                        | `#1E2260` |
| `char`         | Character text hex color                         | `#FFFFFF` |

Default colors sourced from the SBB blue palette (`#2D327D` / `#1E2260`).

## Flip Animation

- On load, all tiles start blank (space character).
- JS reads `line1`…`lineN`, pads/truncates each to `cols` chars.
- Each tile independently cycles through the character sequence until it reaches its target.
- Character set: `A–Z`, `0–9`, space, punctuation (`. , : / -`).
- Flip mechanic (authentic Solari):
  1. Top half snaps instantly to the new character.
  2. Bottom half of old character rotates away (rotateX 0°→90°, pivot at top edge).
  3. Bottom half of new character rotates in (rotateX -90°→0°, pivot at top edge).
  — All via CSS 3D `rotateX` transforms; no canvas, no sprites.
- Each tile gets a random initial delay (`0–800ms / speed`) and random flip speed (`80–140ms / speed`) to mimic real board randomness.

## Sound

- Web Audio API synthesizes a short noise burst (12ms, high-pass filtered) on every flip.
- **Muted by default.**
- Mute toggle button (Bootstrap Icons) fixed bottom-right.

## UI Controls

Two small round icon buttons fixed to the bottom-right corner:
- **`?` (help)** — toggles a popover listing all URL parameters with an example.
- **Speaker icon (mute)** — toggles sound on/off.

## Implementation

- Single `index.html`, no build step.
- CSS 3D transforms for flip animation.
- Web Audio API for click sound (synthesized, no audio files).
- Bootstrap Icons via CDN for UI icons.
- Inter Bold via Google Fonts.

## Edge Cases

- Unsupported characters → replaced with space.
- Text longer than `cols` → truncated.
- Text shorter than `cols` → right-padded with spaces.
- Missing `lineN` params → blank row.
- Invalid hex colors → fall back to defaults.
- Invalid `rows`/`cols`/`speed` → fall back to defaults.
- Characters not in CHAR_SET (e.g. `"`, `!`, accented letters) → space.
