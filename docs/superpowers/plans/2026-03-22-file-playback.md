# File Playback Feature Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add file upload and typewriter playback capability to 墨境 (Mo Jing) writing app

**Architecture:** Single-file app modification, adding playback UI overlay, file reader, and character-by-character text insertion with existing typewriter effects

**Tech Stack:** FileReader API, requestAnimationFrame, existing Web Audio integration

---

## File Structure

**Modify only:** `index.html`

**New code sections:**
- CSS for playback control bar and editor lock overlay
- HTML for upload button and playback controls
- JavaScript for playback engine (~200-300 lines)

**Existing code reused:**
- `showTypeEffect()` - typewriter animation
- `playRetroKeystroke()` - typing sounds
- `updateWordCount()` - word count updates
- `hideWelcome()` - welcome overlay hiding

---

### Task 1: Add Upload Button HTML and CSS

**Files:**
- Modify: `index.html:276` (before "新建" button)

- [ ] **Step 1: Add upload button HTML**

Insert before "新建" button:
```html
<button id="upload-btn" title="上传文件">📁 上传</button>
```

- [ ] **Step 2: Add hidden file input**

Insert at end of `#app` div (before closing tag):
```html
<input type="file" id="file-input" accept=".txt,.md,.json,.csv" hidden>
```

- [ ] **Step 3: Test**

Open page in browser, verify upload button appears in toolbar

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add upload button and hidden file input"
```

---

### Task 2: Add Playback Control Bar HTML

**Files:**
- Modify: `index.html:280` (inside `#editor-wrap`, before `#editor`)

- [ ] **Step 1: Add playback control bar HTML**

Insert:
```html
<div id="playback-bar" style="display:none;">
  <div class="playback-controls">
    <button id="pb-start">⏮</button>
    <button id="pb-rewind">⏪</button>
    <button id="pb-play">▶</button>
    <button id="pb-pause" style="display:none;">⏸</button>
    <button id="pb-forward">⏩</button>
    <button id="pb-end">⏭</button>
    <button id="pb-stop">⏹</button>
    <input type="range" id="pb-speed" min="0.5" max="3" step="0.5" value="1" title="播放速度">
    <span class="pb-speed-label">1x</span>
  </div>
</div>
```

- [ ] **Step 2: Test**

Open page, verify control bar is hidden by default (use devtools to check it exists in DOM)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add playback control bar HTML"
```

---

### Task 3: Add Playback Control Bar CSS

**Files:**
- Modify: `index.html:188` (after `#welcome.hidden` CSS, before `body.focus-mode` CSS)

- [ ] **Step 1: Add playback bar CSS**

```css
#playback-bar {
  position: fixed;
  top: 50px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--toolbar-bg);
  backdrop-filter: blur(10px);
  padding: 8px 16px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
  z-index: 90;
  transition: opacity 0.3s ease;
}
#playback-bar.hidden { display: none !important; }

.playback-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

#playback-bar button {
  background: var(--btn-bg);
  border: 1px solid var(--btn-border);
  color: var(--text);
  width: 32px;
  height: 32px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
}
#playback-bar button:hover { background: var(--btn-hover); }

#pb-speed {
  width: 80px;
  margin: 0 4px;
}

.pb-speed-label {
  font-size: 12px;
  min-width: 24px;
  text-align: center;
}
```

- [ ] **Step 2: Test**

Verify control bar styles are applied (manually set `display:flex` temporarily)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add playback control bar CSS"
```

---

### Task 4: Add Editor Lock Overlay CSS

**Files:**
- Modify: `index.html:220` (after playback bar CSS)

- [ ] **Step 1: Add editor lock overlay CSS**

```css
#editor-lock {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 80;
}
#editor-lock.active { display: flex; }
#editor-lock span {
  background: var(--toolbar-bg);
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 14px;
  opacity: 0.9;
}

#editor-wrap {
  position: relative;
}
```

- [ ] **Step 2: Add lock overlay HTML**

Inside `#editor-wrap`, after `#welcome`:
```html
<div id="editor-lock"><span>播放中，暂停后可编辑</span></div>
```

- [ ] **Step 3: Test**

Verify overlay exists but is hidden (use devtools)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add editor lock overlay CSS and HTML"
```

---

### Task 5: Add Playback State Variables

**Files:**
- Modify: `index.html:300` (after stubs section, before `// === Theme System`)

- [ ] **Step 1: Add playback state variables**

```javascript
// === Playback System ===
let playState = 'idle'; // 'idle' | 'playing' | 'paused' | 'stopped'
let playSpeed = 1;
let currentIndex = 0;
let fileContent = '';
let playbackRAF = null;
let lastPlaybackTime = 0;
```

- [ ] **Step 2: Test**

Open browser console, verify all variables are defined

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add playback state variables"
```

---

### Task 6: Add File Upload Handler

**Files:**
- Modify: `index.html:310` (after playback state variables)

- [ ] **Step 1: Add file upload handler**

```javascript
document.getElementById('upload-btn').addEventListener('click', () => {
  document.getElementById('file-input').click();
});

document.getElementById('file-input').addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (!file) return;

  // Size check: 5MB limit
  if (file.size > 5 * 1024 * 1024) {
    showSaveIndicator('⚠ 文件超过 5MB 限制');
    return;
  }

  // Read file
  const reader = new FileReader();
  reader.onload = (event) => {
    const text = event.target.result;
    if (!text || text.length === 0) {
      showSaveIndicator('⚠ 文件为空');
      return;
    }

    // Confirm if editor has content
    const existingText = (editor.innerText || '').trim();
    if (existingText && !confirm('当前有未保存内容，确定要清空并播放新文件吗？')) {
      return;
    }

    // Start playback
    startPlayback(text);
  };
  reader.onerror = () => {
    showSaveIndicator('⚠ 文件读取失败');
  };
  reader.readAsText(file, 'UTF-8');

  // Reset input
  e.target.value = '';
});
```

- [ ] **Step 2: Test**

Click upload button, select a text file, verify handler runs

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add file upload handler with size check"
```

---

### Task 7: Add Start Playback Function

**Files:**
- Modify: `index.html:345` (after upload handler)

- [ ] **Step 1: Add startPlayback function**

```javascript
function startPlayback(text) {
  // Stop any existing playback
  stopPlayback(false);

  fileContent = text;
  currentIndex = 0;
  playState = 'playing';

  // Hide welcome, clear editor
  hideWelcome();
  editor.textContent = '';
  editor.contentEditable = 'false';

  // Show controls
  document.getElementById('playback-bar').style.display = 'flex';
  document.getElementById('editor-lock').classList.add('active');

  // Update button states
  document.getElementById('pb-play').style.display = 'none';
  document.getElementById('pb-pause').style.display = 'flex';

  lastPlaybackTime = performance.now();
  playbackLoop();
}

function playbackLoop() {
  if (playState !== 'playing') return;

  const now = performance.now();
  const interval = 80 / playSpeed; // 80ms base interval divided by speed

  if (now - lastPlaybackTime >= interval) {
    lastPlaybackTime = now;

    // Check if reached end
    if (currentIndex >= fileContent.length) {
      stopPlayback(true);
      return;
    }

    // Insert character
    const char = fileContent[currentIndex];
    if (char === '\n') {
      document.execCommand('insertText', false, '\n');
    } else {
      document.execCommand('insertText', false, char);
    }

    currentIndex++;
    showTypeEffect();
    playRetroKeystroke(char);

    // Update progress
    updatePlaybackProgress();

    // Update word count occasionally
    if (currentIndex % 5 === 0) {
      updateWordCount();
    }
  }

  playbackRAF = requestAnimationFrame(playbackLoop);
}
```

- [ ] **Step 2: Test**

Call `startPlayback('test')` from console, verify playback starts

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add startPlayback and playbackLoop functions"
```

---

### Task 8: Add Stop Playback Function

**Files:**
- Modify: `index.html:390` (after playbackLoop)

- [ ] **Step 1: Add stopPlayback function**

```javascript
function stopPlayback(finished = false) {
  playState = 'stopped';

  if (playbackRAF) {
    cancelAnimationFrame(playbackRAF);
    playbackRAF = null;
  }

  // Hide controls
  document.getElementById('playback-bar').style.display = 'none';
  document.getElementById('editor-lock').classList.remove('active');

  // Unlock editor
  editor.contentEditable = 'true';

  // Reset button states
  document.getElementById('pb-play').style.display = 'flex';
  document.getElementById('pb-pause').style.display = 'none';

  // Update word count
  updateWordCount();

  if (finished) {
    showSaveIndicator('✓ 播放完成');
  }
}
```

- [ ] **Step 2: Test**

Call `startPlayback('test')` then `stopPlayback()`, verify controls hide

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add stopPlayback function"
```

---

### Task 9: Add Pause/Resume Function

**Files:**
- Modify: `index.html:420` (after stopPlayback)

- [ ] **Step 1: Add togglePause function**

```javascript
function togglePause() {
  if (playState === 'playing') {
    playState = 'paused';
    document.getElementById('pb-play').style.display = 'flex';
    document.getElementById('pb-pause').style.display = 'none';
    document.getElementById('editor-lock').classList.remove('active');
    editor.contentEditable = 'true';
  } else if (playState === 'paused') {
    playState = 'playing';
    document.getElementById('pb-play').style.display = 'none';
    document.getElementById('pb-pause').style.display = 'flex';
    document.getElementById('editor-lock').classList.add('active');
    editor.contentEditable = 'false';
    lastPlaybackTime = performance.now();
    playbackLoop();
  }
}
```

- [ ] **Step 2: Test**

Start playback, call `togglePause()` twice, verify pause/resume works

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add togglePause function"
```

---

### Task 10: Add Playback Progress Display

**Files:**
- Modify: `index.html:440` (after togglePause)

- [ ] **Step 1: Add updatePlaybackProgress function**

```javascript
function updatePlaybackProgress() {
  const wordCountEl = document.getElementById('word-count');
  const total = fileContent.length;
  const current = currentIndex;
  wordCountEl.textContent = `播放中 ${current} / ${total} 字`;
}
```

- [ ] **Step 2: Test**

Start playback, verify progress updates in status bar

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add playback progress display"
```

---

### Task 11: Add Playback Control Button Event Listeners

**Files:**
- Modify: `index.html:448` (after updatePlaybackProgress)

- [ ] **Step 1: Add control button listeners**

```javascript
document.getElementById('pb-start').addEventListener('click', () => {
  if (playState !== 'playing' && playState !== 'paused') return; // Only works when playback active
  currentIndex = 0;
  editor.textContent = '';
  if (playState === 'paused') {
    playState = 'playing';
    document.getElementById('pb-play').style.display = 'none';
    document.getElementById('pb-pause').style.display = 'flex';
    lastPlaybackTime = performance.now();
    playbackLoop();
  }
});

document.getElementById('pb-end').addEventListener('click', () => {
  // Insert remaining content instantly before updating index
  const remaining = fileContent.slice(currentIndex);
  document.execCommand('insertText', false, remaining);
  currentIndex = fileContent.length;
  stopPlayback(true);
});

document.getElementById('pb-rewind').addEventListener('click', () => {
  currentIndex = Math.max(0, currentIndex - 100);
  // Rebuild content up to new position
  editor.textContent = '';
  for (let i = 0; i < currentIndex; i++) {
    document.execCommand('insertText', false, fileContent[i]);
  }
  updatePlaybackProgress();
});

document.getElementById('pb-forward').addEventListener('click', () => {
  const oldIndex = currentIndex;
  currentIndex = Math.min(fileContent.length, currentIndex + 100);
  // Insert skipped content instantly
  const textToInsert = fileContent.slice(oldIndex, currentIndex);
  for (let i = 0; i < textToInsert.length; i++) {
    document.execCommand('insertText', false, textToInsert[i]);
  }
  updatePlaybackProgress();
});

document.getElementById('pb-stop').addEventListener('click', () => {
  stopPlayback(false);
});

document.getElementById('pb-play').addEventListener('click', togglePause);
document.getElementById('pb-pause').addEventListener('click', togglePause);

document.getElementById('pb-speed').addEventListener('input', (e) => {
  playSpeed = parseFloat(e.target.value);
  document.querySelector('.pb-speed-label').textContent = playSpeed + 'x';
});
```

- [ ] **Step 2: Test**

Test each control button: start, end, rewind, forward, stop, play/pause, speed slider

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add playback control button event listeners"
```

---

### Task 12: Add Keyboard Shortcuts

**Files:**
- Modify: `index.html:490` (after speed slider listener)

- [ ] **Step 1: Add keyboard shortcuts**

```javascript
document.addEventListener('keydown', (e) => {
  // Only if playback is active and not typing in input
  if (playState === 'idle') {
    if (e.key === 'Escape') {
      document.body.classList.toggle('focus-mode');
    }
    return;
  }

  // Playback shortcuts
  if (e.code === 'Space') {
    e.preventDefault();
    togglePause();
  } else if (e.code === 'ArrowLeft') {
    e.preventDefault();
    currentIndex = Math.max(0, currentIndex - 10);
    editor.textContent = '';
    for (let i = 0; i < currentIndex; i++) {
      document.execCommand('insertText', false, fileContent[i]);
    }
    updatePlaybackProgress();
  } else if (e.code === 'ArrowRight') {
    e.preventDefault();
    const oldIndex = currentIndex;
    currentIndex = Math.min(fileContent.length, currentIndex + 10);
    const textToInsert = fileContent.slice(oldIndex, currentIndex);
    for (let i = 0; i < textToInsert.length; i++) {
      document.execCommand('insertText', false, textToInsert[i]);
    }
    updatePlaybackProgress();
  } else if (e.key === 'Escape') {
    e.preventDefault();
    stopPlayback(false);
  }
});
```

**Note:** Existing Escape key handler at line 412 toggles focus-mode. The new handler above will replace/extend this - integrate the focus-mode toggle into the idle-state check at the top of the new handler, or remove the old listener and add focus-mode toggle to the new handler's idle-state path.

- [ ] **Step 2: Test**

Test each shortcut: Space (pause), Left/Right arrows (skip), Esc (stop)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add keyboard shortcuts for playback"
```

---

### Task 13: Handle Playback Interruption

**Files:**
- Modify: `index.html:650` (existing "New Document" handler)

- [ ] **Step 1: Update new document handler to stop playback**

Find existing `new-btn` click handler and add:
```javascript
// Stop playback if active
if (playState === 'playing' || playState === 'paused') {
  if (!confirm('播放中，确定要新建文档吗？')) {
    return;
  }
  stopPlayback(false);
}
```

Place before the `if (text && !confirm(...))` check.

- [ ] **Step 2: Test**

Start playback, click new button, verify confirmation and stop

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: handle playback interruption on new document"
```

---

### Task 14: Handle Theme Change During Playback

**Files:**
- Modify: `index.html:315` (setTheme function)

- [ ] **Step 1: Update setTheme to not break playback**

Check the existing `setTheme` function (around line 319):
1. If it only modifies CSS variables/DOM attributes → no change needed
2. If it modifies editor content (innerHTML/textContent) → add check to skip during playback:
```javascript
if (playState === 'playing' || playState === 'paused') return;
```
3. If it removes/re-adds event listeners → ensure playback state is preserved

- [ ] **Step 2: Test**

Start playback, switch themes, verify playback continues with new theme

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "test: verify theme change during playback"
```

---

### Task 15: Mobile Responsive CSS

**Files:**
- Modify: `index.html:258` (existing media query for mobile)

- [ ] **Step 1: Add mobile playback bar styles**

Inside existing `@media (max-width: 768px)` block, add:
```css
#playback-bar {
  flex-wrap: wrap;
  justify-content: center;
  top: auto;
  bottom: 70px;
  max-width: 90vw;
}
.playback-controls {
  flex-wrap: wrap;
  justify-content: center;
}
```

- [ ] **Step 2: Test**

Test on mobile or devtools mobile view, verify control bar fits screen

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add mobile responsive playback bar styles"
```

---

### Task 16: Add Accessibility Support

**Files:**
- Modify: `index.html:48` (existing `prefers-reduced-motion` check)

- [ ] **Step 1: Add reduced motion check for playback**

After existing reduced motion check, add:
```javascript
// For playback with reduced motion, skip animations
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
let showTypeEffectOriginal = showTypeEffect;
function showTypeEffectPlayback() {
  if (!prefersReducedMotion || playState !== 'playing') {
    showTypeEffectOriginal();
  }
}
// Use this version during playback loop
```

- [ ] **Step 2: Test**

Enable reduced motion preference, verify no typing animation during playback

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add accessibility support for playback"
```

---

### Task 17: Final Integration and Testing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Test complete workflow**

1. Upload a test file (sample.txt)
2. Verify playback starts automatically
3. Test play/pause toggle
4. Test fast forward/rewind
5. Test skip to start/end
6. Test speed slider
7. Test keyboard shortcuts
8. Test theme change during playback
9. Test new document during playback
10. Test empty file handling
11. Test large file rejection (>5MB)
12. Test completion (auto-stop)

- [ ] **Step 2: Fix any issues found**

Address bugs discovered during testing.

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete file playback feature"
```

---

## Testing Notes

Since this is frontend code without automated tests, manual testing is required:

| Test Case | Expected Result |
|------------|----------------|
| Upload valid text file | Playback starts automatically |
| Upload empty file | "文件为空" message shown |
| Upload file > 5MB | "文件超过 5MB 限制" shown |
| Play button | Resumes playback |
| Pause button | Pauses playback, unlocks editor |
| Fast forward (+100 chars) | Skips ahead instantly |
| Rewind (-100 chars) | Jumps back and replays |
| Skip to end | Inserts remaining content, stops |
| Stop button | Stops playback, keeps content |
| Space key | Toggle play/pause |
| Arrow keys | Skip 10 chars forward/back |
| Esc key | Stop playback |
| Speed slider | Changes playback speed |
| Theme change during playback | Continues with new theme |
| New document during playback | Confirms then stops |
| Playback completes | Auto-stop, unlock editor |

## Browser Compatibility

Test on:
- Chrome 80+
- Safari 14+
- Firefox 78+
- Edge 80+
