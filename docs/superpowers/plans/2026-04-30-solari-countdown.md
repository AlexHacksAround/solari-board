# Solari Countdown Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `countdown.html` that renders an HH:MM:SS split-flap countdown timer with scaling, sound, and a flash+chime finish.

**Architecture:** Self-contained HTML file reusing the tile CSS and flip animation from `index.html`. JS drives a `setInterval` that decrements total seconds, flips only changed digits, and triggers flash+chime at zero. The board is wrapped in a `transform: scale()` container sized to `width`% of the viewport.

**Tech Stack:** HTML5, CSS3 (3D transforms), vanilla JS (ES6+), Web Audio API, Inter Bold via Google Fonts, Bootstrap Icons via CDN.

---

## Chunk 1: HTML skeleton + tile CSS + colon tile

### Task 1: HTML skeleton

**Files:**
- Create: `countdown.html`

- [ ] **Step 1: Create `countdown.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta name="color-scheme" content="dark" />
  <title>Solari Countdown</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@700&display=swap" rel="stylesheet" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css" />
  <style>
    /* CSS here */
  </style>
</head>
<body>
  <div id="scale-wrap">
    <div id="board">
      <!-- tiles injected by JS -->
    </div>
  </div>
  <button id="mute-btn" title="Toggle sound">
    <i class="bi bi-volume-mute-fill"></i>
  </button>
  <script>
    /* JS here */
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify opens in browser with blank dark page, no console errors**

- [ ] **Step 3: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown html skeleton"
```

---

### Task 2: CSS — base layout, tile styles, colon tile

**Files:**
- Modify: `countdown.html` — `<style>` block

- [ ] **Step 1: Replace `/* CSS here */` with base layout and tile styles**

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg:     #888888;
  --charbg: #000000;
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

/* Scale wrapper — JS sets transform on this */
#scale-wrap {
  display: flex;
  align-items: center;
  justify-content: center;
  transform-origin: top center;
}

#board {
  display: flex;
  align-items: center;
  gap: 3px;
}

/* --- Digit tile (48×64px, split-flap) --- */
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

.tile::after {
  content: '';
  position: absolute;
  left: 0; right: 0;
  top: calc(50% - 1px);
  height: 2px;
  background: var(--bg);
  z-index: 20;
  pointer-events: none;
}

.tile-half {
  position: absolute;
  left: 0; right: 0;
  height: 32px;
  overflow: hidden;
  color: var(--char);
  font-size: 28px;
  font-weight: 700;
  font-family: 'Inter', sans-serif;
  text-align: center;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}

.tile-top {
  top: 0;
  line-height: 64px;
}

.tile-bottom {
  top: 32px;
  line-height: 0px;
}

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

/* --- Colon tile (static, no split line) --- */
.colon {
  width: 24px;
  height: 64px;
  background: var(--charbg);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--char);
  font-size: 28px;
  font-weight: 700;
  font-family: 'Inter', sans-serif;
}

/* --- Flash animation at zero --- */
.tile-wrapper.flash {
  visibility: hidden;
}

/* --- Mute button --- */
#mute-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: none;
  background: rgba(255,255,255,0.2);
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

#mute-btn:hover {
  background: rgba(255,255,255,0.35);
}
```

- [ ] **Step 2: Manually verify: temporarily add a tile+colon to `#board` in HTML, confirm styling looks correct, then remove**

- [ ] **Step 3: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown tile CSS and colon tile"
```

---

## Chunk 2: URL params, board construction, scaling

### Task 3: URL param parsing + color application

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Replace `/* JS here */` with param parsing**

```js
const DEFAULTS = {
  hh: 0, mm: 0, ss: 0,
  width: 75,
  background: '#888888', charbg: '#000000', char: '#FFFFFF'
};

function getParams() {
  const p = new URLSearchParams(location.search);

  function posInt(key) {
    const v = parseInt(p.get(key));
    return (Number.isInteger(v) && v >= 0) ? v : DEFAULTS[key];
  }

  const width = parseFloat(p.get('width'));

  return {
    hh:         posInt('hh'),
    mm:         posInt('mm'),
    ss:         posInt('ss'),
    width:      (isFinite(width) && width >= 10 && width <= 100) ? width : DEFAULTS.width,
    background: validHex(p.get('background')) || DEFAULTS.background,
    charbg:     validHex(p.get('charbg'))     || DEFAULTS.charbg,
    char:       validHex(p.get('char'))        || DEFAULTS.char,
  };
}

function validHex(val) {
  if (!val) return null;
  const v = val.startsWith('#') ? val : '#' + val;
  return /^#[0-9A-Fa-f]{6}$/.test(v) ? v : null;
}

function applyColors(params) {
  const s = document.documentElement.style;
  s.setProperty('--bg',     params.background);
  s.setProperty('--charbg', params.charbg);
  s.setProperty('--char',   params.char);
}
```

- [ ] **Step 2: Verify in browser console**

```js
// Expected: { hh:0, mm:10, ss:0, width:75, background:'#888888', ... }
// open countdown.html?mm=10 and run getParams()
```

- [ ] **Step 3: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown URL param parsing"
```

---

### Task 4: Tile factory, board construction, scaling

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Add tile factory and board builder**

Append to JS:

```js
function createDigitTile(digit) {
  const wrapper = document.createElement('div');
  wrapper.className = 'tile-wrapper';

  const tile = document.createElement('div');
  tile.className = 'tile';

  const top = document.createElement('div');
  top.className = 'tile-top tile-half';
  top.textContent = digit;

  const bot = document.createElement('div');
  bot.className = 'tile-bottom tile-half';
  bot.textContent = digit;

  tile.appendChild(top);
  tile.appendChild(bot);
  wrapper.appendChild(tile);
  return { wrapper, tile, top, bot, current: digit };
}

function createColon() {
  const el = document.createElement('div');
  el.className = 'colon';
  el.textContent = ':';
  return el;
}

// Returns array of 6 tile objects: [h1, h2, m1, m2, s1, s2]
function buildBoard(totalSeconds) {
  const board = document.getElementById('board');
  board.innerHTML = '';

  const { h1, h2, m1, m2, s1, s2 } = splitDigits(totalSeconds);
  const digits = [h1, h2, m1, m2, s1, s2];
  const tiles = [];

  digits.forEach((d, i) => {
    const t = createDigitTile(String(d));
    board.appendChild(t.wrapper);
    tiles.push(t);
    if (i === 1 || i === 3) board.appendChild(createColon());
  });

  return tiles;
}

function splitDigits(totalSecs) {
  const hh = Math.floor(totalSecs / 3600);
  const mm = Math.floor((totalSecs % 3600) / 60);
  const ss = totalSecs % 60;
  return {
    h1: Math.floor(hh / 10), h2: hh % 10,
    m1: Math.floor(mm / 10), m2: mm % 10,
    s1: Math.floor(ss / 10), s2: ss % 10,
  };
}
```

- [ ] **Step 2: Add scaling logic**

Append to JS:

```js
// Natural board width: 6 digit tiles (48px) + 2 colons (24px) + gaps (3px × 7)
const NATURAL_BOARD_WIDTH = 6 * 48 + 2 * 24 + 7 * 3; // = 357px

function applyScale(widthPct) {
  const factor = (window.innerWidth * widthPct / 100) / NATURAL_BOARD_WIDTH;
  document.getElementById('scale-wrap').style.transform = `scale(${factor})`;
}
```

- [ ] **Step 3: Wire up DOMContentLoaded (no animation yet)**

Append to JS:

```js
document.addEventListener('DOMContentLoaded', () => {
  const params = getParams();
  applyColors(params);

  const totalSeconds = params.hh * 3600 + params.mm * 60 + params.ss;
  const tiles = buildBoard(totalSeconds);

  applyScale(params.width);
  window.addEventListener('resize', () => applyScale(params.width));

  console.log('board built, tiles:', tiles.length, 'totalSeconds:', totalSeconds);
});
```

- [ ] **Step 4: Verify in browser**

Open `countdown.html?hh=0&mm=10&ss=0`. Expected:
- Board shows `00:10:00` (static, no animation yet)
- Board fills ~75% of viewport width
- Resize window → board rescales

- [ ] **Step 5: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown board construction and viewport scaling"
```

---

## Chunk 3: Flip animation + countdown logic

### Task 5: Flip engine (adapted from index.html)

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Add flip function**

Append to JS (before DOMContentLoaded):

```js
function flipTile(tileObj, nextChar, flipMs) {
  return new Promise(resolve => {
    // Snap top half immediately
    tileObj.top.textContent = nextChar;

    // Old bottom flips away
    tileObj.bot.classList.add('tile-flip-out');
    tileObj.bot.style.setProperty('--flip-ms', flipMs + 'ms');

    // New bottom flips in
    const newBot = document.createElement('div');
    newBot.className = 'tile-bottom tile-half tile-flip-in';
    newBot.style.setProperty('--flip-ms', flipMs + 'ms');
    newBot.textContent = nextChar;
    tileObj.tile.appendChild(newBot);

    setTimeout(() => {
      tileObj.bot.classList.remove('tile-flip-out');
      tileObj.bot.textContent = nextChar;
      newBot.remove();
      tileObj.current = nextChar;
      resolve();
    }, flipMs + 16);
  });
}
```

- [ ] **Step 2: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown flip animation engine"
```

---

### Task 6: Countdown tick logic

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Add tick function**

Append to JS (before DOMContentLoaded):

```js
const FLIP_MS = 120;

function updateDisplay(tiles, prevDigits, nextDigits) {
  const keys = ['h1','h2','m1','m2','s1','s2'];
  keys.forEach((key, i) => {
    const next = String(nextDigits[key]);
    if (next !== String(prevDigits[key])) {
      playClick();
      flipTile(tiles[i], next, FLIP_MS);
    }
  });
}
```

- [ ] **Step 2: Update DOMContentLoaded to start countdown**

Replace the existing DOMContentLoaded listener with:

```js
document.addEventListener('DOMContentLoaded', () => {
  const params = getParams();
  applyColors(params);

  let remaining = params.hh * 3600 + params.mm * 60 + params.ss;
  const tiles = buildBoard(remaining);

  applyScale(params.width);
  window.addEventListener('resize', () => applyScale(params.width));

  // Setup mute button
  const btn = document.getElementById('mute-btn');
  const icon = btn.querySelector('i');
  btn.addEventListener('click', () => {
    muted = !muted;
    icon.className = muted ? 'bi bi-volume-mute-fill' : 'bi bi-volume-up-fill';
    if (!muted) getAudioCtx().resume();
  });

  if (remaining <= 0) {
    triggerFinish(tiles);
    return;
  }

  let prevDigits = splitDigits(remaining);

  const interval = setInterval(() => {
    remaining--;
    const nextDigits = splitDigits(remaining);
    updateDisplay(tiles, prevDigits, nextDigits);
    prevDigits = nextDigits;

    if (remaining <= 0) {
      clearInterval(interval);
      setTimeout(() => triggerFinish(tiles), FLIP_MS + 50);
    }
  }, 1000);
});
```

- [ ] **Step 3: Verify in browser**

Open `countdown.html?mm=1`. Expected:
- Shows `00:01:00`, counts down each second
- Only changed digits flip (seconds digits flip every second, minutes digit flips at :59→:58→... etc.)
- No finish behavior yet

- [ ] **Step 4: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown tick logic with selective digit flipping"
```

---

## Chunk 4: Sound + finish behavior

### Task 7: Web Audio — tick click and finish chime

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Add audio engine**

Append to JS (before DOMContentLoaded):

```js
let audioCtx = null;
let muted = true;

function getAudioCtx() {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  return audioCtx;
}

function playClick() {
  if (muted) return;
  const ctx = getAudioCtx();
  const bufferSize = Math.floor(ctx.sampleRate * 0.012);
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
  const data = buffer.getChannelData(0);
  for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

  const source = ctx.createBufferSource();
  source.buffer = buffer;

  const gain = ctx.createGain();
  gain.gain.setValueAtTime(0.35, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.012);

  const filter = ctx.createBiquadFilter();
  filter.type = 'highpass';
  filter.frequency.value = 1200;

  source.connect(filter);
  filter.connect(gain);
  gain.connect(ctx.destination);
  source.start();
}

function playChime() {
  if (muted) return;
  const ctx = getAudioCtx();
  [880, 440].forEach((freq, i) => {
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.frequency.value = freq;
    osc.type = 'sine';
    const start = ctx.currentTime + i * 0.35;
    gain.gain.setValueAtTime(0.4, start);
    gain.gain.exponentialRampToValueAtTime(0.001, start + 0.3);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start(start);
    osc.stop(start + 0.35);
  });
}
```

- [ ] **Step 2: Verify click plays on digit flip**

Open `countdown.html?mm=1`, unmute with the button. Expected: audible tick on each digit change.

- [ ] **Step 3: Commit**

```bash
git add countdown.html
git commit -m "feat: add Web Audio tick click and finish chime"
```

---

### Task 8: Finish behavior — flash + chime + click-to-dismiss

**Files:**
- Modify: `countdown.html` — `<script>` block

- [ ] **Step 1: Add triggerFinish function**

Append to JS (before DOMContentLoaded):

```js
function triggerFinish(tiles) {
  playChime();

  let visible = true;
  const flashInterval = setInterval(() => {
    visible = !visible;
    tiles.forEach(t => {
      t.wrapper.classList.toggle('flash', !visible);
    });
  }, 500);

  function dismiss() {
    clearInterval(flashInterval);
    tiles.forEach(t => t.wrapper.classList.remove('flash'));
    document.removeEventListener('click', dismiss);
  }

  document.addEventListener('click', dismiss);
}
```

- [ ] **Step 2: Verify finish behavior**

Open `countdown.html?ss=3`. Expected:
- Counts down 3→2→1→0
- At zero: all tiles flash every 500ms, chime plays (if unmuted)
- Click anywhere: flashing stops, `00:00:00` shows static

- [ ] **Step 3: Verify zero-start edge case**

Open `countdown.html` (no params). Expected:
- Shows `00:00:00` static immediately, no countdown, no flash (already at zero)

- [ ] **Step 4: Commit**

```bash
git add countdown.html
git commit -m "feat: add countdown finish behavior - flash, chime, click-to-dismiss"
```

---

## Chunk 5: Final polish

### Task 9: Edge case hardening + final verification

**Files:**
- Modify: `countdown.html` — minor

- [ ] **Step 1: Verify all edge cases**

| URL | Expected |
|-----|----------|
| `countdown.html` | `00:00:00` static (already zero) |
| `countdown.html?mm=10` | `00:10:00` counting down |
| `countdown.html?hh=1&mm=30` | `01:30:00` counting down |
| `countdown.html?ss=3&width=50` | Board fills 50% of viewport |
| `countdown.html?ss=3&background=000000&charbg=1a1a1a&char=FF8800` | Black/amber theme |
| `countdown.html?hh=-1&mm=abc` | Falls back to `00:00:00` |
| `countdown.html?width=200` | Clamped to 100% |

- [ ] **Step 2: Final visual check**

Open `countdown.html?mm=1&width=75`. Expected: clean split-flap countdown, correct scaling, tick sound on unmute, chime + flash at zero.

- [ ] **Step 3: Commit**

```bash
git add countdown.html
git commit -m "feat: solari countdown complete"
git push
```
