# Solari Display Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` that renders an animated split-flap Solari display with two rows of 20 tiles, driven entirely by URL parameters.

**Architecture:** One self-contained HTML file with embedded CSS and JS. CSS 3D `rotateX` transforms handle the flip animation per tile. JS reads URL params on load, normalises text to 20 chars, then drives each tile through a character cycle with random delays and speeds.

**Tech Stack:** HTML5, CSS3 (3D transforms, custom properties), vanilla JS (ES6+), Inter Bold via Google Fonts.

---

## Chunk 1: Static tile layout

### Task 1: HTML skeleton + page styling

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with document skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta name="color-scheme" content="dark" />
  <title>Solari Display</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@700&display=swap" rel="stylesheet" />
  <style>
    /* CSS goes here in Task 2 */
  </style>
</head>
<body>
  <div id="board">
    <div class="row" id="row1"></div>
    <div class="row" id="row2"></div>
  </div>
  <script>
    /* JS goes here in Task 3+ */
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify file opens in browser with blank page (no errors in console)**

Open `index.html` in a browser. Expected: blank page, no console errors.

- [ ] **Step 3: Commit**

```bash
git init  # only if repo not yet initialised
git add index.html
git commit -m "feat: add html skeleton"
```

---

### Task 2: Tile CSS

**Files:**
- Modify: `index.html` — `<style>` block

- [ ] **Step 1: Add CSS custom properties and base styles**

Replace the `/* CSS goes here */` comment with:

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg:     #2D327D;
  --charbg: #1E2260;
  --char:   #FFFFFF;
}

body {
  background: var(--bg);
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-family: 'Inter', sans-serif;
}

#board {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.row {
  display: flex;
  gap: 3px;
}
```

- [ ] **Step 2: Add tile styles**

Append inside `<style>`:

```css
/* Wrap each tile in a perspective container so 3D transforms don't clip oddly */
.tile-wrapper {
  perspective: 200px;
  perspective-origin: center center;
}

.tile {
  width: 48px;
  height: 64px;
  position: relative;
  background: var(--charbg);
  border-radius: 4px;
  overflow: hidden;
  transform-style: flat;
}

/* horizontal split gap — sits on top of everything */
.tile::after {
  content: '';
  position: absolute;
  left: 0; right: 0;
  top: calc(50% - 1px);
  height: 2px;
  background: var(--bg);
  z-index: 10;
  pointer-events: none;
}

/* Each half clips to its own 50% of the tile */
.tile-half {
  position: absolute;
  left: 0; right: 0;
  height: 32px;           /* exactly half of 64px */
  overflow: hidden;
  display: flex;
  justify-content: center;
  color: var(--char);
  font-size: 28px;
  font-weight: 700;
  font-family: 'Inter', sans-serif;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}

/* Top half: shows upper portion of character */
.tile-top {
  top: 0;
  align-items: flex-end;  /* character baseline flush with the split */
}

/* Bottom half: shows lower portion of character */
.tile-bottom {
  top: 32px;
  align-items: flex-start; /* character top flush with the split */
}

/*
  Real Solari mechanic:
  - Top half of NEW char appears instantly (no animation)
  - Bottom half of OLD char rotates DOWN and away (flip-out)
  - Bottom half of NEW char rotates DOWN into place (flip-in)
  Both flip-out and flip-in pivot from the TOP edge of the bottom half.
*/
.tile-flip-out {
  transform-origin: top center;
  animation: flip-out var(--flip-ms, 120ms) linear forwards;
}

.tile-flip-in {
  transform-origin: top center;
  animation: flip-in var(--flip-ms, 120ms) linear forwards;
}

@keyframes flip-out {
  from { transform: rotateX(0deg); }
  to   { transform: rotateX(90deg); }
}

@keyframes flip-in {
  from { transform: rotateX(-90deg); }
  to   { transform: rotateX(0deg); }
}
```

- [ ] **Step 3: Manually verify by adding a test tile in HTML**

Temporarily add inside `#row1`:

```html
<div class="tile-wrapper">
  <div class="tile">
    <div class="tile-top tile-half">A</div>
    <div class="tile-bottom tile-half">A</div>
  </div>
</div>
```

Open browser. Expected: one dark blue tile on a blue background, showing the top half of "A" above the split line and the bottom half below it — character visually centred across the split.

- [ ] **Step 4: Remove test tile, commit**

```bash
git add index.html
git commit -m "feat: add tile CSS with split-flap styling"
```

---

## Chunk 2: URL parameter parsing + tile generation

### Task 3: URL param parsing

**Files:**
- Modify: `index.html` — `<script>` block

- [ ] **Step 1: Write param-parsing logic**

Replace `/* JS goes here */` with:

```js
const CHAR_SET = ' ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789.,:/- ';
const COLS = 20;
const DEFAULTS = { background: '#2D327D', charbg: '#1E2260', char: '#FFFFFF' };

function getParams() {
  const p = new URLSearchParams(location.search);
  return {
    line1:  normalise(p.get('line1') || ''),
    line2:  normalise(p.get('line2') || ''),
    background: validHex(p.get('background')) || DEFAULTS.background,
    charbg:     validHex(p.get('charbg'))     || DEFAULTS.charbg,
    char:       validHex(p.get('char'))        || DEFAULTS.char,
  };
}

function normalise(text) {
  return text
    .toUpperCase()
    .split('')
    .map(c => CHAR_SET.includes(c) ? c : ' ')
    .slice(0, COLS)
    .join('')
    .padEnd(COLS, ' ');
}

function validHex(val) {
  if (!val) return null;
  const v = val.startsWith('#') ? val : '#' + val;
  return /^#[0-9A-Fa-f]{6}$/.test(v) ? v : null;
}
```

- [ ] **Step 2: Apply colours to CSS custom properties**

Append to JS:

```js
function applyColors(params) {
  const root = document.documentElement.style;
  root.setProperty('--bg',     params.background);
  root.setProperty('--charbg', params.charbg);
  root.setProperty('--char',   params.char);
  // also update body background which uses var(--bg)
}
```

- [ ] **Step 3: Manually test in browser console**

Open `index.html`. In console run:

```js
normalise('hello world!');
// Expected: "HELLO WORLD!        " (20 chars, unsupported ! → space)
validHex('2D327D');   // Expected: '#2D327D'
validHex('ZZZZZZ');   // Expected: null
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add URL param parsing and colour application"
```

---

### Task 4: Tile DOM generation

**Files:**
- Modify: `index.html` — `<script>` block

- [ ] **Step 1: Add tile factory function**

Append to JS:

```js
function createTile(char) {
  const wrapper = document.createElement('div');
  wrapper.className = 'tile-wrapper';

  const tile = document.createElement('div');
  tile.className = 'tile';

  const top = document.createElement('div');
  top.className = 'tile-top tile-half';
  top.textContent = char;

  const bot = document.createElement('div');
  bot.className = 'tile-bottom tile-half';
  bot.textContent = char;

  tile.appendChild(top);
  tile.appendChild(bot);
  wrapper.appendChild(tile);
  return { wrapper, tile, top, bot };
}
```

- [ ] **Step 2: Add row-building function**

Append to JS:

```js
function buildRow(rowEl, text) {
  rowEl.innerHTML = '';
  const tiles = [];
  for (const char of text) {
    const t = createTile(' ');   // start blank
    rowEl.appendChild(t.wrapper);
    tiles.push({ ...t, target: char, current: ' ' });
  }
  return tiles;
}
```

- [ ] **Step 3: Wire up on DOMContentLoaded**

Append to JS:

```js
document.addEventListener('DOMContentLoaded', () => {
  const params = getParams();
  applyColors(params);
  const row1Tiles = buildRow(document.getElementById('row1'), params.line1);
  const row2Tiles = buildRow(document.getElementById('row2'), params.line2);
  // animation called in Task 5
  console.log('tiles built', row1Tiles.length, row2Tiles.length);
});
```

- [ ] **Step 4: Verify in browser**

Open `index.html?line1=HELLO&line2=WORLD`. Expected: 20 blank tiles per row, console logs "tiles built 20 20".

Open `index.html?line1=HELLO&line2=WORLD&background=000000`. Expected: black background.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: generate tile DOM from URL params"
```

---

## Chunk 3: Flip animation engine

### Task 5: Per-tile flip stepper

**Files:**
- Modify: `index.html` — `<script>` block

- [ ] **Step 1: Add flip-step function**

Append to JS (before the DOMContentLoaded listener):

```js
function getNextChar(current) {
  const idx = CHAR_SET.indexOf(current);
  return CHAR_SET[(idx + 1) % CHAR_SET.length];
}

function flipToNext(tileObj, flipMs) {
  return new Promise(resolve => {
    const next = getNextChar(tileObj.current);

    /*
     * Real Solari mechanic:
     * 1. Top half snaps immediately to show top of NEW char
     * 2. Old bottom half rotates away (flip-out: 0→90deg from top edge)
     * 3. New bottom half rotates in  (flip-in: -90→0deg from top edge)
     */

    // Step 1: snap top half to new char immediately
    tileObj.top.textContent = next;

    // Step 2: old bottom flips away
    tileObj.bot.classList.add('tile-flip-out');
    tileObj.bot.style.setProperty('--flip-ms', flipMs + 'ms');

    // Step 3: new bottom half flips in, starts behind the tile
    const newBot = document.createElement('div');
    newBot.className = 'tile-bottom tile-half tile-flip-in';
    newBot.style.setProperty('--flip-ms', flipMs + 'ms');
    newBot.textContent = next;
    tileObj.tile.appendChild(newBot);

    // After animation completes, clean up and commit state
    setTimeout(() => {
      tileObj.bot.classList.remove('tile-flip-out');
      tileObj.bot.textContent = next;
      newBot.remove();
      tileObj.current = next;
      resolve();
    }, flipMs + 16);  // +16ms buffer for browser frame scheduling
  });
}
```

- [ ] **Step 2: Add tile animator (cycles until target)**

Append to JS:

```js
async function animateTile(tileObj, delay) {
  await new Promise(r => setTimeout(r, delay));
  while (tileObj.current !== tileObj.target) {
    const flipMs = 80 + Math.random() * 60;  // 80–140ms
    await flipToNext(tileObj, flipMs);
  }
}
```

- [ ] **Step 3: Launch all tile animations**

Append to JS (a new `startAnimation` function):

```js
function startAnimation(allTiles) {
  for (const t of allTiles) {
    const delay = Math.random() * 800;  // 0–800ms
    animateTile(t, delay);
  }
}
```

- [ ] **Step 4: Call startAnimation from DOMContentLoaded**

Update the listener to pass tiles:

```js
document.addEventListener('DOMContentLoaded', () => {
  const params = getParams();
  applyColors(params);
  const row1Tiles = buildRow(document.getElementById('row1'), params.line1);
  const row2Tiles = buildRow(document.getElementById('row2'), params.line2);
  startAnimation([...row1Tiles, ...row2Tiles]);
});
```

- [ ] **Step 5: Test in browser**

Open `index.html?line1=HELLO%20WORLD&line2=TRACK%2003`. Expected:
- All tiles start blank
- Tiles begin flipping with visible randomness in timing
- Each tile lands on its target character
- Split-flap flip animation visible on each tile
- No console errors

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add flip animation engine with random delay and speed"
```

---

## Chunk 4: Polish + edge cases

### Task 6: Edge case hardening + final verification

**Files:**
- Modify: `index.html` — minor tweaks

- [ ] **Step 1: Verify edge cases in browser**

Test each URL in browser and confirm expected output:

| URL | Expected |
|-----|----------|
| `index.html` | 2 rows of 20 blank tiles, flip to blank |
| `index.html?line1=ABCDEFGHIJKLMNOPQRSTUVWXYZ` | Truncated to 20 chars |
| `index.html?line1=HI` | "HI" + 18 blank tiles |
| `index.html?line1=hello!world` | "HELLO WORLD" (! → space) |
| `index.html?background=ZZZZZZ` | Falls back to `#2D327D` |
| `index.html?background=FF0000&charbg=000000&char=FFFF00` | Red page, black tiles, yellow text |

- [ ] **Step 2: Add board gap styling for visual breathing room**

Inside `<style>`, append:

```css
body {
  padding: 48px 24px;
}
```

*(Merge with existing `body` rule if present — add `padding` property.)*

- [ ] **Step 3: Final visual check**

Open `index.html?line1=ZURICH+HB&line2=PLATFORM+7&background=2D327D&charbg=1E2260&char=FFFFFF`.

Expected: Deep blue board, white letters, clean flip animation landing on "ZURICH HB" / "PLATFORM 7".

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: solari display complete - flip animation, URL params, configurable colors"
```
