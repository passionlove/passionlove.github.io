# Typewriter Writing App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file static writing app with typewriter effects, ambient background music, switchable themes, and Markdown export.

**Architecture:** Single `index.html` with embedded CSS and JS. No build tools, no external dependencies. Audio via Web Audio API (typing sounds) and Base64-embedded loops (ambient music). State persisted in localStorage.

**Tech Stack:** HTML5, CSS3 (custom properties for theming), vanilla JavaScript, Web Audio API

**Spec:** `docs/superpowers/specs/2026-03-22-typewriter-writing-app-design.md`

---

## File Map

| File | Responsibility |
|------|---------------|
| `index.html` | Entire application: HTML structure, CSS styles, JavaScript logic, embedded audio data |
| `CNAME` | Custom domain config (existing, untouched) |

Since this is a single-file app, the plan is organized by logical component. Each task builds on the previous and produces a working (if incomplete) page that can be opened in a browser to verify.

---

### Task 1: HTML Structure & Base CSS

**Files:**
- Create: `index.html` (replaces existing mahjong game)

Scaffold the page skeleton with all HTML elements, base CSS, and essential JS stubs that later tasks will flesh out. This avoids forward-reference errors during incremental testing.

- [ ] **Step 1: Write the HTML skeleton**

Create `index.html` with:
- `<!DOCTYPE html>`, lang="zh-CN", meta charset/viewport
- `<title>` — "写作" (keep it minimal)
- `<style>` block (empty for now)
- `<div id="app">` containing:
  - `<header id="toolbar">` — with buttons: theme switcher (`#theme-btn`), music controls (`#music-btn`, `#music-volume`, `#type-volume`), typewriter style (`#style-btn`), export (`#export-btn`), new document (`#new-btn`)
  - `<main id="editor-wrap">` containing `<div id="editor" contenteditable="true" spellcheck="false">`
  - `<footer id="statusbar">` — word count `<span id="word-count">` and saved indicator `<span id="save-indicator">`
- `<script>` block (empty for now)

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>写作</title>
  <style>/* Step 2 */</style>
</head>
<body>
  <div id="app">
    <header id="toolbar">
      <div class="toolbar-group">
        <button id="theme-btn" title="切换主题">🎨 深夜</button>
        <button id="style-btn" title="打字机风格">⌨ 复古机械</button>
      </div>
      <div class="toolbar-group">
        <button id="music-btn" title="氛围音乐">🔇 静谧</button>
        <label class="vol-label">氛围 <input type="range" id="music-volume" min="0" max="100" value="50"></label>
        <label class="vol-label">按键 <input type="range" id="type-volume" min="0" max="100" value="50"></label>
      </div>
      <div class="toolbar-group">
        <button id="export-btn" title="导出 Markdown" disabled>📥 导出</button>
        <button id="new-btn" title="新建文档">📄 新建</button>
      </div>
    </header>
    <main id="editor-wrap">
      <div id="editor" contenteditable="true" spellcheck="false"></div>
    </main>
    <footer id="statusbar">
      <span id="word-count">0 字</span>
      <span id="save-indicator"></span>
    </footer>
  </div>
  <script>/* Step 3+ */</script>
</body>
</html>
```

- [ ] **Step 2: Write base CSS**

In the `<style>` block, add:

```css
/* Reset */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

/* CSS custom properties — Deep Night theme as default */
:root {
  --bg: linear-gradient(135deg, #0a0e27 0%, #1a1a3e 100%);
  --text: #f0e6d3;
  --accent: #6b7fd7;
  --toolbar-bg: rgba(10, 14, 39, 0.85);
  --statusbar-bg: rgba(10, 14, 39, 0.7);
  --editor-bg: transparent;
}

html, body { height: 100%; overflow: hidden; }
body {
  background: var(--bg);
  color: var(--text);
  font-family: -apple-system, "Microsoft YaHei", sans-serif;
  display: flex;
  flex-direction: column;
}

/* Toolbar: hidden by default, appears on hover */
#toolbar {
  position: fixed;
  top: 0; left: 0; right: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 20px;
  background: var(--toolbar-bg);
  backdrop-filter: blur(10px);
  z-index: 100;
  transform: translateY(-100%);
  transition: transform 0.3s ease;
}
#toolbar:hover, #toolbar.visible { transform: translateY(0); }
/* Hover trigger zone at top of viewport — JS handles the hover logic */
#app::before {
  content: '';
  position: fixed;
  top: 0; left: 0; right: 0;
  height: 30px;
  z-index: 99;
}

.toolbar-group { display: flex; align-items: center; gap: 8px; }

#toolbar button {
  background: rgba(255,255,255,0.1);
  border: 1px solid rgba(255,255,255,0.2);
  color: var(--text);
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  transition: background 0.2s;
}
#toolbar button:hover { background: rgba(255,255,255,0.2); }
#toolbar button:disabled { opacity: 0.4; cursor: default; }

.vol-label {
  color: var(--text);
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
  opacity: 0.8;
}
.vol-label input[type="range"] { width: 60px; }

/* Editor */
#editor-wrap {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 80px 20px 60px;
  overflow-y: auto;
}
#editor {
  width: 100%;
  max-width: 700px;
  min-height: 60vh;
  outline: none;
  font-size: 18px;
  line-height: 1.8;
  color: var(--text);
  caret-color: var(--text);
  white-space: pre-wrap;
  word-wrap: break-word;
}

/* Status bar */
#statusbar {
  position: fixed;
  bottom: 0; left: 0; right: 0;
  display: flex;
  justify-content: space-between;
  padding: 6px 20px;
  background: var(--statusbar-bg);
  font-size: 12px;
  opacity: 0.6;
  transition: opacity 0.3s;
}
#statusbar:hover { opacity: 1; }
#save-indicator { transition: opacity 0.5s; }

/* Focus mode */
body.focus-mode #toolbar,
body.focus-mode #statusbar { display: none !important; }
body.focus-mode #editor-wrap { padding: 40px 20px; }

/* Mobile */
@media (max-width: 768px) {
  #editor-wrap { padding: 60px 16px 80px; }
  #editor { max-width: 100%; font-size: 16px; }
  #toolbar {
    top: auto; bottom: 0;
    transform: translateY(100%);
    flex-wrap: wrap;
    gap: 8px;
  }
  #toolbar:hover, #toolbar.visible { transform: translateY(0); }
  #statusbar { bottom: auto; top: 0; }
}
```

Note: The `#app::before` zone is styled but hover detection is handled purely in JS (Task 2, Step 3).

- [ ] **Step 3: Add JS stubs for forward references**

In the `<script>` block, add stubs that later tasks will replace with full implementations. This prevents `ReferenceError` during incremental testing:

```javascript
// === Stubs (replaced by later tasks) ===
let currentMusicScene = 3; // silence default
function savePrefs() {}
function scheduleSave() {}
```

- [ ] **Step 4: Verify in browser**

Open `index.html` in a browser. Verify:
- Full viewport dark background
- Toolbar hidden, appears when hovering near top
- Editor area centered, contenteditable works (can type)
- Status bar visible at bottom
- Mobile layout works (resize window)

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: scaffold HTML structure and base CSS for writing app"
```

---

### Task 2: Theme System

**Files:**
- Modify: `index.html` — add theme CSS variables and JS switching logic

- [ ] **Step 1: Add theme CSS variables**

In the `<style>` block, after the `:root` (Deep Night default), add theme data-attribute selectors:

```css
[data-theme="sunset"] {
  --bg: linear-gradient(135deg, #2d1b4e 0%, #4a2040 50%, #7a3b2e 100%);
  --text: #f5e6b8;
  --accent: #e8a87c;
  --toolbar-bg: rgba(45, 27, 78, 0.85);
  --statusbar-bg: rgba(45, 27, 78, 0.7);
}
[data-theme="dawn"] {
  --bg: linear-gradient(135deg, #faf6f0 0%, #f5ede0 100%);
  --text: #2c2c2c;
  --accent: #7a9e7e;
  --toolbar-bg: rgba(250, 246, 240, 0.9);
  --statusbar-bg: rgba(250, 246, 240, 0.8);
}
[data-theme="ocean"] {
  --bg: linear-gradient(135deg, #0c2340 0%, #0d4f6b 50%, #1a7a7a 100%);
  --text: #d4f0f0;
  --accent: #4ecdc4;
  --toolbar-bg: rgba(12, 35, 64, 0.85);
  --statusbar-bg: rgba(12, 35, 64, 0.7);
}
```

- [ ] **Step 2: Write theme switching JS**

In the `<script>` block:

```javascript
const THEMES = [
  { id: 'night', label: '深夜', attr: '' },
  { id: 'sunset', label: '晚霞', attr: 'sunset' },
  { id: 'dawn', label: '晨曦', attr: 'dawn' },
  { id: 'ocean', label: '海洋', attr: 'ocean' }
];
const THEME_MUSIC_MAP = { night: 'rain', sunset: 'fire', dawn: 'forest', ocean: 'silence' };

let currentThemeIndex = 0;

function setTheme(index) {
  currentThemeIndex = index;
  const theme = THEMES[index];
  if (theme.attr) {
    document.documentElement.setAttribute('data-theme', theme.attr);
  } else {
    document.documentElement.removeAttribute('data-theme');
  }
  document.getElementById('theme-btn').textContent = '🎨 ' + theme.label;
}

document.getElementById('theme-btn').addEventListener('click', () => {
  setTheme((currentThemeIndex + 1) % THEMES.length);
  savePrefs();
});
```

- [ ] **Step 3: Add toolbar hover trigger (JS fallback)**

```javascript
// Toolbar hover trigger — JS fallback for cross-browser support
const toolbar = document.getElementById('toolbar');
const triggerZone = 30; // px from top
document.addEventListener('mousemove', (e) => {
  if (e.clientY <= triggerZone) {
    toolbar.classList.add('visible');
  }
});
toolbar.addEventListener('mouseleave', () => {
  toolbar.classList.remove('visible');
});
```

- [ ] **Step 4: Verify in browser**

Open page. Click theme button repeatedly — should cycle through Deep Night → Sunset → Dawn → Ocean → Deep Night. Background gradient, text color, and toolbar style should all change.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add 4-theme system with CSS variable switching"
```

---

### Task 3: Typewriter Style System (Visual Only)

**Files:**
- Modify: `index.html` — add typewriter style CSS and switching JS

- [ ] **Step 1: Add typewriter style CSS**

```css
/* Typewriter styles */
[data-typestyle="retro"] #editor {
  font-family: "Courier New", "Noto Serif SC", monospace;
  font-weight: bold;
  letter-spacing: 0.5px;
}
[data-typestyle="retro"] #editor { caret-color: transparent; }
[data-typestyle="retro"] #editor::after {
  /* Block cursor simulation is handled in JS */
}

[data-typestyle="modern"] #editor {
  font-family: -apple-system, "Microsoft YaHei", "PingFang SC", sans-serif;
}

[data-typestyle="minimal"] #editor {
  font-family: inherit;
}

/* Retro stamp animation — applied to a transient overlay element */
@keyframes stamp-in {
  0% { transform: scale(1.4) translateY(-2px); opacity: 0.6; }
  100% { transform: scale(1) translateY(0); opacity: 1; }
}

/* Modern fade animation */
@keyframes fade-char {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

/* Retro cursor: blinking block */
.retro-cursor {
  display: inline-block;
  width: 0.6em;
  height: 1.1em;
  background: var(--text);
  vertical-align: text-bottom;
  animation: blink-block 1s step-end infinite;
}
@keyframes blink-block {
  50% { opacity: 0; }
}

/* Modern cursor: breathing gradient blink */
[data-typestyle="modern"] #editor {
  caret-color: var(--accent);
}
@keyframes breathing-caret {
  0%, 100% { caret-color: var(--accent); }
  50% { caret-color: transparent; }
}
[data-typestyle="modern"] #editor:focus {
  animation: breathing-caret 1.5s ease-in-out infinite;
}
```

- [ ] **Step 2: Write style switching JS**

```javascript
const STYLES = [
  { id: 'retro', label: '复古机械' },
  { id: 'modern', label: '现代柔和' },
  { id: 'minimal', label: '极简' }
];
let currentStyleIndex = 0;

function setTypeStyle(index) {
  currentStyleIndex = index;
  const style = STYLES[index];
  document.documentElement.setAttribute('data-typestyle', style.id);
  document.getElementById('style-btn').textContent = '⌨ ' + style.label;
}

// Respect prefers-reduced-motion
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  currentStyleIndex = 2; // Minimal
}
setTypeStyle(currentStyleIndex);

document.getElementById('style-btn').addEventListener('click', () => {
  setTypeStyle((currentStyleIndex + 1) % STYLES.length);
  savePrefs();
});
```

- [ ] **Step 3: Verify in browser**

Click style button — font should change between Courier (retro), sans-serif (modern), and system default (minimal).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add 3 typewriter visual styles with prefers-reduced-motion"
```

---

### Task 4: Editor Core — Input Handling, Word Count, Paste, Focus Mode

**Files:**
- Modify: `index.html` — editor JS logic

- [ ] **Step 1: Write editor core JS**

```javascript
const editor = document.getElementById('editor');
const wordCountEl = document.getElementById('word-count');
const exportBtn = document.getElementById('export-btn');

// Word count
function updateWordCount() {
  const text = editor.innerText || '';
  // Chinese: count each character. For mixed text, count Chinese chars + English words
  const chineseChars = (text.match(/[\u4e00-\u9fff\u3400-\u4dbf]/g) || []).length;
  const englishWords = (text.match(/[a-zA-Z]+/g) || []).length;
  const total = chineseChars + englishWords;
  wordCountEl.textContent = total + ' 字';
  exportBtn.disabled = text.trim().length === 0;
}

// Paste: strip to plain text
editor.addEventListener('paste', (e) => {
  e.preventDefault();
  const text = e.clipboardData.getData('text/plain');
  document.execCommand('insertText', false, text);
});

// Input event — single consolidated handler (Tasks 8 and 11 will add to onInput)
function onInput(e) {
  updateWordCount();
  scheduleSave();
}
editor.addEventListener('input', (e) => onInput(e));

// Focus mode toggle
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    document.body.classList.toggle('focus-mode');
  }
});

// Note: editor.focus() is called in the initialization block (Task 11)
```

- [ ] **Step 2: Verify in browser**

- Type text → word count updates
- Paste rich text → should be plain text only
- Press Esc → toolbar and status bar disappear (focus mode)
- Press Esc again → they return

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add editor input handling, word count, paste strip, focus mode"
```

---

### Task 5: localStorage Persistence

**Files:**
- Modify: `index.html` — auto-save and restore JS

- [ ] **Step 1: Write persistence JS (replaces stubs from Task 1)**

Replace the stub `savePrefs()` and `scheduleSave()` functions with full implementations:

```javascript
const saveIndicator = document.getElementById('save-indicator');
let saveTimeout = null;

function scheduleSave() {
  clearTimeout(saveTimeout);
  saveTimeout = setTimeout(saveToStorage, 1000);
}

function saveToStorage() {
  const data = {
    text: editor.innerText || '',
    theme: currentThemeIndex,
    style: currentStyleIndex,
    musicScene: currentMusicScene || 3, // default silence
    musicVolume: document.getElementById('music-volume').value,
    typeVolume: document.getElementById('type-volume').value
  };
  try {
    localStorage.setItem('writer-data', JSON.stringify(data));
    showSaveIndicator();
  } catch (e) {
    // Storage full — try saving just text
    try {
      localStorage.setItem('writer-data', JSON.stringify({ text: data.text }));
      showSaveIndicator('⚠ 存储空间不足，仅保存文本');
    } catch (e2) {
      showSaveIndicator('⚠ 保存失败');
    }
  }
}

function showSaveIndicator(msg) {
  saveIndicator.textContent = msg || '✓ 已保存';
  saveIndicator.style.opacity = '1';
  setTimeout(() => { saveIndicator.style.opacity = '0'; }, 2000);
}

function savePrefs() {
  scheduleSave();
}

function restoreFromStorage() {
  try {
    const raw = localStorage.getItem('writer-data');
    if (!raw) return;
    const data = JSON.parse(raw);
    if (data.text) {
      editor.textContent = data.text;
      updateWordCount();
    }
    if (data.theme !== undefined) setTheme(data.theme);
    if (data.style !== undefined) setTypeStyle(data.style);
    if (data.musicVolume !== undefined) document.getElementById('music-volume').value = data.musicVolume;
    if (data.typeVolume !== undefined) document.getElementById('type-volume').value = data.typeVolume;
    // musicScene restored in Task 7 when audio is ready
    // Cursor to end
    const range = document.createRange();
    const sel = window.getSelection();
    range.selectNodeContents(editor);
    range.collapse(false);
    sel.removeAllRanges();
    sel.addRange(range);
  } catch (e) { /* corrupt data, ignore */ }
}
```

- [ ] **Step 2: Call restoreFromStorage on page load**

Add at the end of the `<script>` block (temporary — will be reorganized in Task 11):
```javascript
restoreFromStorage();
```

- [ ] **Step 3: Verify in browser**

- Type some text, wait 1 second → "✓ 已保存" appears
- Refresh page → text is restored, cursor at end
- Switch theme, refresh → theme is restored

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add localStorage auto-save with debounce and full state restore"
```

---

### Task 6: Typing Sound Effects (Web Audio API)

**Files:**
- Modify: `index.html` — audio engine JS

- [ ] **Step 1: Write audio engine**

```javascript
let audioCtx = null;
let typingGain = null;
let musicGain = null;

function initAudio() {
  if (audioCtx) return;
  try {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    typingGain = audioCtx.createGain();
    typingGain.gain.value = document.getElementById('type-volume').value / 100;
    typingGain.connect(audioCtx.destination);
    musicGain = audioCtx.createGain();
    musicGain.gain.value = document.getElementById('music-volume').value / 100;
    musicGain.connect(audioCtx.destination);
  } catch (e) {
    audioCtx = null; // Audio disabled
  }
}

// Ensure audio init on first user interaction
document.addEventListener('click', initAudio, { once: true });
document.addEventListener('keydown', initAudio, { once: true });

// Volume controls
document.getElementById('type-volume').addEventListener('input', (e) => {
  if (typingGain) typingGain.gain.value = e.target.value / 100;
  savePrefs();
});
document.getElementById('music-volume').addEventListener('input', (e) => {
  if (musicGain) musicGain.gain.value = e.target.value / 100;
  savePrefs();
});
```

- [ ] **Step 2: Write typing sound synthesizers**

```javascript
function playRetroKeystroke(key) {
  if (!audioCtx || !typingGain || currentStyleIndex === 2) return; // Minimal = no sound
  const now = audioCtx.currentTime;
  const pitchVariation = 1 + (Math.random() - 0.5) * 0.1; // ±5%

  if (currentStyleIndex === 0) {
    // Retro mechanical: noise burst + high-freq click
    const bufferSize = audioCtx.sampleRate * 0.05; // 50ms
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const data = buffer.getChannelData(0);
    const isEnter = key === 'Enter';
    const isBackspace = key === 'Backspace';
    const amplitude = isEnter ? 0.5 : isBackspace ? 0.2 : 0.35;
    for (let i = 0; i < bufferSize; i++) {
      const decay = Math.exp(-i / (bufferSize * (isEnter ? 0.4 : 0.2)));
      data[i] = (Math.random() * 2 - 1) * amplitude * decay;
    }
    const source = audioCtx.createBufferSource();
    source.buffer = buffer;
    source.playbackRate.value = pitchVariation;
    source.connect(typingGain);
    source.start(now);
  } else if (currentStyleIndex === 1) {
    // Modern soft: gentle sine click
    const osc = audioCtx.createOscillator();
    const oscGain = audioCtx.createGain();
    const isEnter = key === 'Enter';
    osc.type = 'sine';
    osc.frequency.value = (isEnter ? 600 : 800) * pitchVariation;
    oscGain.gain.setValueAtTime(0.15, now);
    oscGain.gain.exponentialRampToValueAtTime(0.001, now + 0.03);
    osc.connect(oscGain);
    oscGain.connect(typingGain);
    osc.start(now);
    osc.stop(now + 0.03);
  }
}
```

- [ ] **Step 3: Wire typing sound to keydown event**

```javascript
editor.addEventListener('keydown', (e) => {
  playRetroKeystroke(e.key);
});
```

- [ ] **Step 4: Verify in browser**

- Type in Retro style → hear crisp mechanical clicks
- Switch to Modern → hear soft gentle clicks
- Switch to Minimal → no sound
- Adjust typing volume slider → volume changes
- Press Enter → distinct sound variant

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Web Audio API typing sound synthesis for retro and modern styles"
```

---

### Task 7: Background Ambient Music

**Files:**
- Modify: `index.html` — ambient audio system

Since we need Base64 audio but can't include real recordings in the plan, we'll generate ambient sounds programmatically using Web Audio API as well. The sounds will be generated once into an AudioBuffer and looped. This avoids the need for external audio files while maintaining the architecture described in the spec.

- [ ] **Step 1: Write ambient sound generators**

```javascript
const SCENES = [
  { id: 'rain', label: '🌧 雨夜' },
  { id: 'fire', label: '🔥 壁炉' },
  { id: 'forest', label: '🌲 森林' },
  { id: 'silence', label: '🔇 静谧' }
];
let currentMusicScene = 3; // silence default (already declared as stub in Task 1, now used by this module)
let ambientSource = null;

function generateAmbientBuffer(sceneId) {
  if (!audioCtx) return null;
  const sr = audioCtx.sampleRate;
  const duration = 6; // 6 seconds loop
  const length = sr * duration;
  const buffer = audioCtx.createBuffer(1, length, sr);
  const data = buffer.getChannelData(0);

  if (sceneId === 'rain') {
    // Brown noise (filtered random) for rain
    let last = 0;
    for (let i = 0; i < length; i++) {
      const white = Math.random() * 2 - 1;
      last = (last + (0.02 * white)) / 1.02;
      data[i] = last * 3.5;
      // Occasional louder "drops"
      if (Math.random() < 0.0001) {
        const dropLen = Math.min(sr * 0.05, length - i);
        for (let j = 0; j < dropLen; j++) {
          data[i + j] += (Math.random() * 2 - 1) * 0.3 * Math.exp(-j / (sr * 0.01));
        }
      }
    }
  } else if (sceneId === 'fire') {
    // Crackling: random bursts of noise
    for (let i = 0; i < length; i++) {
      let sample = 0;
      // Base warm hum
      sample += Math.sin(i / sr * Math.PI * 2 * 60) * 0.05;
      // Random crackles
      if (Math.random() < 0.003) {
        const crackleLen = Math.min(Math.floor(sr * 0.02 * Math.random()), length - i);
        for (let j = 0; j < crackleLen; j++) {
          data[i + j] = (data[i + j] || 0) + (Math.random() * 2 - 1) * 0.4 * Math.exp(-j / (sr * 0.005));
        }
      }
      data[i] = (data[i] || 0) + sample;
    }
  } else if (sceneId === 'forest') {
    // Wind: slowly modulated noise
    let last = 0;
    for (let i = 0; i < length; i++) {
      const white = Math.random() * 2 - 1;
      last = (last + (0.01 * white)) / 1.01;
      const windMod = 0.5 + 0.5 * Math.sin(i / sr * Math.PI * 2 * 0.15);
      data[i] = last * 2.5 * windMod;
      // Random bird chirp
      if (Math.random() < 0.00005) {
        const chirpLen = Math.min(Math.floor(sr * 0.1), length - i);
        for (let j = 0; j < chirpLen; j++) {
          const freq = 2000 + 1000 * Math.sin(j / sr * Math.PI * 2 * 8);
          data[i + j] += Math.sin(j / sr * Math.PI * 2 * freq) * 0.1 * Math.exp(-j / (sr * 0.04));
        }
      }
    }
  }
  return buffer;
}

function playAmbient(sceneId) {
  stopAmbient();
  if (!audioCtx || sceneId === 'silence') return;
  const buffer = generateAmbientBuffer(sceneId);
  if (!buffer) return;
  ambientSource = audioCtx.createBufferSource();
  ambientSource.buffer = buffer;
  ambientSource.loop = true;
  ambientSource.connect(musicGain);
  ambientSource.start();
}

function stopAmbient() {
  if (ambientSource) {
    try { ambientSource.stop(); } catch(e) {}
    ambientSource = null;
  }
}

function setMusicScene(index) {
  currentMusicScene = index;
  const scene = SCENES[index];
  document.getElementById('music-btn').textContent = scene.label;
  if (audioCtx) {
    playAmbient(scene.id);
  }
}

document.getElementById('music-btn').addEventListener('click', () => {
  initAudio();
  setMusicScene((currentMusicScene + 1) % SCENES.length);
  savePrefs();
});
```

- [ ] **Step 2: Restore music scene from localStorage**

In `restoreFromStorage()`, after the music volume restore line, add:
```javascript
if (data.musicScene !== undefined) {
  currentMusicScene = data.musicScene;
  // Don't auto-play — wait for user interaction due to browser policy
  document.getElementById('music-btn').textContent = SCENES[currentMusicScene].label;
}
```

- [ ] **Step 3: Verify in browser**

- Click music button → cycles through Rain → Fire → Forest → Silence
- Rain: hear continuous textured noise
- Fire: hear crackling with warm base
- Forest: hear wind with occasional chirps
- Silence: no sound
- Adjust music volume slider → ambient volume changes

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add procedural ambient sound generation (rain, fire, forest)"
```

---

### Task 8: Typewriter Visual Animation

**Files:**
- Modify: `index.html` — typing animation effects

- [ ] **Step 1: Write animation overlay system**

Rather than wrapping characters in spans, use a transient floating element that appears at the cursor position and fades away:

```javascript
function getCaretCoordinates() {
  const sel = window.getSelection();
  if (!sel.rangeCount) return null;
  const range = sel.getRangeAt(0).cloneRange();
  range.collapse(false);
  const rect = range.getClientRects()[0];
  // Fallback: if rect is null (empty line), use editor position
  if (!rect) {
    const editorRect = editor.getBoundingClientRect();
    return { x: editorRect.left, y: editorRect.top };
  }
  return { x: rect.left, y: rect.top };
}

function showTypeEffect() {
  if (currentStyleIndex === 2) return; // Minimal: no effect
  const pos = getCaretCoordinates();
  if (!pos) return;

  const el = document.createElement('div');
  el.className = 'type-effect';
  el.style.left = pos.x + 'px';
  el.style.top = pos.y + 'px';

  if (currentStyleIndex === 0) {
    // Retro: stamp flash
    el.classList.add('type-effect-retro');
    // Random jitter
    el.style.transform = `translate(${(Math.random()-0.5)*3}px, ${(Math.random()-0.5)*2}px)`;
  } else {
    // Modern: soft glow
    el.classList.add('type-effect-modern');
  }

  document.body.appendChild(el);
  el.addEventListener('animationend', () => el.remove());
}

// Add to onInput() function (defined in Task 4):
// After updateWordCount() and scheduleSave(), add:
if (e.inputType === 'insertText' || e.inputType === 'insertParagraph') {
  requestAnimationFrame(showTypeEffect);
}
```

Update the `onInput` function in Task 4's code to include this check.

- [ ] **Step 2: Add animation CSS**

```css
.type-effect {
  position: fixed;
  pointer-events: none;
  z-index: 50;
}
.type-effect-retro {
  width: 14px;
  height: 22px;
  background: var(--text);
  opacity: 0.6;
  animation: stamp-flash 0.12s ease-out forwards;
}
@keyframes stamp-flash {
  0% { opacity: 0.6; transform: scale(1.3); }
  100% { opacity: 0; transform: scale(1); }
}

.type-effect-modern {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--accent);
  opacity: 0.5;
  animation: soft-glow 0.3s ease-out forwards;
}
@keyframes soft-glow {
  0% { opacity: 0.5; transform: scale(1); }
  100% { opacity: 0; transform: scale(2.5); }
}
```

- [ ] **Step 3: Verify in browser**

- Type in Retro style → see brief stamp flash at cursor position with slight jitter
- Type in Modern style → see soft expanding glow dot at cursor
- Type in Minimal → no visual effect
- Paste text → no per-character effects (instant)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add transient typing animation effects (retro stamp, modern glow)"
```

---

### Task 9: Markdown Export

**Files:**
- Modify: `index.html` — export function

- [ ] **Step 1: Write export function**

```javascript
document.getElementById('export-btn').addEventListener('click', () => {
  const text = editor.innerText || '';
  if (!text.trim()) return;

  // Generate filename from first line
  const firstLine = text.split('\n')[0].trim();
  let filename;
  if (firstLine) {
    // Sanitize: remove illegal filename chars, truncate
    filename = firstLine
      .replace(/[\/\\:*?"<>|]/g, '')
      .substring(0, 50)
      .trim();
    filename = filename || '写作';
  } else {
    const d = new Date();
    filename = `写作-${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
  }
  filename += '.md';

  const blob = new Blob([text], { type: 'text/markdown;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
});
```

- [ ] **Step 2: Verify in browser**

- Type "我的第一篇文章\n这是正文" → click export → downloads `我的第一篇文章.md`
- Clear text, type starting with newline → downloads `写作-2026-03-22.md`
- Empty editor → export button should be disabled (greyed out)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Markdown export with sanitized filename"
```

---

### Task 10: New Document & Theme-Music Pairing

**Files:**
- Modify: `index.html` — new document button and theme-music default pairing

- [ ] **Step 1: Write new document handler**

```javascript
document.getElementById('new-btn').addEventListener('click', () => {
  const text = (editor.innerText || '').trim();
  if (text && !confirm('当前文档未导出，确定要新建吗？')) return;
  editor.textContent = '';
  updateWordCount();
  saveToStorage();
  editor.focus();
});
```

- [ ] **Step 2: Add theme-music default pairing**

Update the `setTheme` function to accept an optional `fromUserClick` parameter. Only auto-pair music on manual theme button clicks, NOT during restore:

```javascript
// Modify setTheme to accept a source parameter:
function setTheme(index, fromUserClick) {
  currentThemeIndex = index;
  const theme = THEMES[index];
  if (theme.attr) {
    document.documentElement.setAttribute('data-theme', theme.attr);
  } else {
    document.documentElement.removeAttribute('data-theme');
  }
  document.getElementById('theme-btn').textContent = '🎨 ' + theme.label;
  // Only auto-pair music on manual clicks, not on restore
  if (fromUserClick) {
    const pairedMusic = THEME_MUSIC_MAP[theme.id];
    const pairedIndex = SCENES.findIndex(s => s.id === pairedMusic);
    if (pairedIndex !== -1 && pairedIndex !== currentMusicScene) {
      setMusicScene(pairedIndex);
    }
  }
}

// Update theme button click handler:
document.getElementById('theme-btn').addEventListener('click', () => {
  setTheme((currentThemeIndex + 1) % THEMES.length, true);
  savePrefs();
});
```

- [ ] **Step 3: Verify in browser**

- Type text → click New → confirmation dialog appears → confirm → editor clears
- Switch theme to Sunset → music should auto-switch to Fireplace
- User can still manually switch music independent of theme

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add new document button and theme-music default pairing"
```

---

### Task 11: Final Integration & Polish

**Files:**
- Modify: `index.html` — ensure all pieces work together, fix initialization order

- [ ] **Step 1: Set correct initialization order**

Ensure the script block has this order at the bottom:
```javascript
// Initialization
setTheme(0);       // Default theme
setTypeStyle(0);   // Default style (or 2 if reduced-motion)
restoreFromStorage(); // Override defaults with saved prefs
updateWordCount();
editor.focus();
```

- [ ] **Step 2: Add page title update and AudioContext resume**

```javascript
// Update page title with first line of text (or default)
function updateTitle() {
  const text = (editor.innerText || '').split('\n')[0].trim();
  document.title = text ? text.substring(0, 30) + ' — 写作' : '写作';
}
// Add updateTitle() call inside onInput() function

// Resume AudioContext when tab regains focus (especially needed for iOS Safari)
document.addEventListener('visibilitychange', () => {
  if (!document.hidden && audioCtx && audioCtx.state === 'suspended') {
    audioCtx.resume();
  }
});
```

Add `updateTitle()` to the `onInput` function alongside `updateWordCount` and `scheduleSave`.

- [ ] **Step 3: Remove old mahjong game code**

The old `index.html` is fully replaced in Task 1. Verify no remnants of the old game exist.

- [ ] **Step 4: Full verification**

Open in browser and verify the complete flow:
1. Page loads → dark background, cursor ready in editor
2. Type text → word count updates, typing sound plays, visual effect appears
3. Wait 1s → "✓ 已保存" appears
4. Switch theme → colors change, music changes
5. Switch typewriter style → font and effects change
6. Click export → .md file downloads
7. Refresh → all state restored
8. Press Esc → focus mode (clean, no UI)
9. Resize to mobile → layout adapts
10. Test in Chrome and Safari

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: final integration and polish for typewriter writing app"
```

---

### Task 12: Delete old game docs (cleanup)

**Files:**
- Delete: any old game-related files if they exist beyond `index.html` (which was already replaced)

- [ ] **Step 1: Verify clean state**

```bash
git status
```

Ensure only `index.html`, `CNAME`, and `docs/` directory exist. No stale files.

- [ ] **Step 2: Final commit**

```bash
git add -A
git commit -m "chore: clean up repository for writing app"
```
