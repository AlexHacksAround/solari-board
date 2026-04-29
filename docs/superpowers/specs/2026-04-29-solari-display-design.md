# Solari Display — Design Spec
_2026-04-29_

## Overview

A single self-contained `index.html` that renders an animated split-flap (Solari) display in the browser. No build tools, no dependencies beyond Inter Bold via Google Fonts.

## Layout

- Two rows of **20 character tiles** each, centered on the page.
- Tile size: **48×64px**, Inter Bold ~28px.
- A thin 1–2px gap between top and bottom halves of each tile gives the authentic mechanical look.

## URL Parameters

| Parameter    | Description                        | Default   |
|--------------|------------------------------------|-----------|
| `line1`      | Text for row 1 (max 20 chars)      | 20 spaces |
| `line2`      | Text for row 2 (max 20 chars)      | 20 spaces |
| `background` | Page background hex color          | `#2D327D` |
| `charbg`     | Tile background hex color          | `#1E2260` |
| `char`       | Character text hex color           | `#FFFFFF` |

Default colors sourced from `svg-druck/01_Pfeile/01_Pfeilrichtungen/1000_Pfeil-links-blau.svg`.

## Flip Animation

- On load, all tiles start as blank (space).
- JS reads `line1`/`line2`, pads/truncates to 20 chars.
- Each tile cycles through the character sequence until it reaches its target.
- Character set: `A–Z`, `0–9`, space, punctuation (`. , : / -`).
- Flip mechanic: top half stays static, bottom half rotates down (0°→90°, old char), then new top half rotates in (90°→0°) — CSS 3D `rotateX` transforms.
- Each tile gets a random delay (0–800ms) and random flip speed (80–140ms per flip) to mimic real board randomness.
- All tiles flip simultaneously, landing at staggered times.

## Implementation

- CSS 3D transforms (no canvas, no sprites).
- Single `index.html`, no build step.

## Edge Cases

- Unsupported characters → replaced with space.
- Text > 20 chars → truncated.
- Text < 20 chars → right-padded with spaces.
- Missing params → blank display.
- Invalid hex colors → fall back to defaults.
