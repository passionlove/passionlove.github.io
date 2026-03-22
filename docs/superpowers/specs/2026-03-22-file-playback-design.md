# File Playback Feature - Design Spec

## Overview

Add file playback capability to the 墨境 (Mo Jing) writing app. Users can upload text files and watch the content appear character-by-character with typewriter effects, creating an immersive reading experience.

Existing features remain unchanged; playback is an additive feature.

## Core Use Case

Reading experience — users want to see text appear gradually as if being typed in real-time, providing a calm, focused way to consume text content.

## UI Layout

### Upload Button
- Location: Toolbar, before the "New" (新建) button
- Label: "📁 上传"
- Behavior: Opens hidden `<input type="file">` for file selection

### Playback Control Bar
- Appears when playback starts, floating above the editor
- Semi-transparent background matching theme
- Contains:
  - ⏮ Skip to start
  - ⏪ Rewind (10 seconds or 100 characters)
  - ▶ Play / ⏸ Pause
  - ⏩ Forward (10 seconds or 100 characters)
  - ⏭ Skip to end
  - ⏹ Stop (clears played content)
  - Speed slider (0.5x - 3x)
- Disappears when playback stops

### Status Bar Extension
- During playback: shows progress "播放中 1234 / 5000 字"
- Uses existing status bar layout

### Editor Lock
- During playback: `contenteditable="false"` on editor
- Semi-transparent overlay with message "播放中，暂停后可编辑"
- Unlocked when paused or stopped

## Playback Control Logic

### Core Variables
- `playState`: 'idle' | 'playing' | 'paused' | 'stopped'
- `playSpeed`: Current playback speed multiplier (default 1x)
- `currentIndex`: Character index of current playback position
- `fileContent`: Full text content of uploaded file

### Character Timing
- Base interval: 80ms per character (normal typing speed)
- Actual interval = base interval / playSpeed
- Uniform handling: Chinese characters, punctuation, spaces all treated same

### Skip Forward/Backward
- Forward: `currentIndex += 100`, insert skipped characters instantly (no typing animation)
- Backward: `currentIndex -= 100`, clear editor and replay to new position

### Pause/Resume
- Driven by `requestAnimationFrame` for smooth playback
- Paused state saves current position, resume continues from same point

## File Upload Handling

### File Selection
- Hidden `<input type="file">` triggered by button click
- `accept=".txt,.md,.json,.csv"` as hint (but not enforced)

### File Reading
- `FileReader.readAsText()` with UTF-8 encoding
- File size limit: 5MB (prevent large files from causing lag)
- No external dependencies

### Content Validation
- Check for text MIME type or content type
- Binary content detected: show error message in status bar

### Playback Trigger
- On successful read:
  - If editor has content: show confirmation dialog
  - Clear editor (if confirmed)
  - Hide welcome overlay
  - Show playback control bar
  - Auto-start playback from beginning

### Error Handling
- File read failure: error message in status bar
- File too large: "文件超过 5MB 限制" message

## Typewriter Effect Integration

### Reuse Existing Effects
- Playback respects current typewriter style (Retro / Modern / Minimal)
- Retro: `type-effect-retro` animation + mechanical keystroke sound
- Modern: `type-effect-modern` animation + soft keystroke sound
- Minimal: text insertion only, no animation or sound

### Character Insertion
- Use `document.execCommand('insertText', false, char)` to maintain undo history
- After each character insert, call existing `showTypeEffect()`
- Update word count every ~5 characters via `updateWordCount()`

### Special Characters
- Newline: insert `<br>` or trigger enter effect
- Space: normal insertion
- Punctuation: same as normal characters

### Performance
- Batch insert: skipped characters (fast forward/rewind) inserted without animation/sound
- `requestAnimationFrame` for smooth playback

## State Management & Persistence

### No Playback Persistence
- Playback state NOT saved to localStorage
- File content NOT auto-saved to localStorage
- Normal save only triggered after user manually modifies editor

### Playback Interruption
- User clicks "New": pause, confirm, then clear
- User uploads new file: pause current, confirm, then replace
- Page refresh/close: stop playback, no confirmation

### Edge Cases
- Playback reaches end: auto-stop, unlock editor, hide control bar
- Empty file: show "文件为空" message, don't start playback
- Theme change during playback: continue with real-time new theme styles

### Keyboard Shortcuts
- Space: Play/Pause
- Left Arrow: Rewind 10 characters
- Right Arrow: Forward 10 characters
- Esc: Stop playback and exit

## Accessibility

- `prefers-reduced-motion`: Auto-select Minimal style during playback
- Control bar buttons: accessible labels and keyboard navigation
- Focus management: Upload button in tab order

## Browser Compatibility

- Same as existing: Chrome 80+, Safari 14+, Firefox 78+, Edge 80+
- FileReader API required (widely supported)

## Implementation Notes

- Single file `index.html` modification
- No external dependencies
- ~200-300 lines of new code expected
- Existing code paths mostly unchanged
