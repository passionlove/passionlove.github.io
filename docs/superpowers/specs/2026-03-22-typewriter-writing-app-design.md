# Typewriter Writing App - Design Spec

## Overview

A single-page static writing application with a typewriter aesthetic, ambient background music, and atmosphere-driven themes. Targeted at writers who want an immersive, distraction-free writing experience where the act of typing itself is enjoyable.

Deployed on GitHub Pages at `agiwork.top`. Single `index.html` file, zero external dependencies.

## Core Positioning

Atmosphere-focused writing experience. Visual and auditory ambiance is the priority — the writing environment should feel like a place you want to be.

## Page Layout

- Full viewport (100vh/100vw), no scrollbar distractions
- **Top toolbar**: semi-transparent, hidden by default, appears on mouse hover or keyboard shortcut. Contains: theme switcher, music controls, typewriter style selector, export button, new document button
- **Center writing area**: max-width 700px, vertically centered, simulates a paper/typing surface
- **Bottom status bar**: semi-transparent fade-in, shows word count and "saved" indicator
- **Esc key**: toggles full focus mode (hides all UI, text only)

## Typewriter Effect System

Three switchable styles:

### 1. Retro Mechanical
- Characters appear with slight random offset (1-2px jitter) simulating uneven metal typebar strikes
- "Stamp down" animation on character appearance
- Bold monospace serif font (Courier style)
- Thick solid block cursor, steady blink
- Sound: crisp mechanical keystroke (high-frequency decay + noise pulse, ~50ms)

### 2. Modern Soft
- Characters fade in (opacity 0→1, ~150ms)
- Clean sans-serif font (system Chinese font)
- Thin line cursor, breathing gradient blink
- Sound: gentle "click" (low-frequency sine wave + fast decay, ~30ms)

### 3. Minimal
- No animation, characters appear instantly
- System default font
- Standard cursor
- No typing sound effects

All styles: keystroke pitch varies randomly (±5%) to avoid repetitive feel. Enter and Backspace have distinct sound variants.

## Background Ambient Music

4 ambient scenes, using embedded Base64 audio loops for realism:

| Scene | Description |
|-------|-------------|
| Rainy Night | Rain sounds + occasional low thunder rumble |
| Fireplace | Crackling fire pulses + warm low-frequency hum |
| Forest | Wind (bandpass noise with cyclic modulation) + random bird chirps |
| Silence | No background audio, typing sounds only |

## Audio Architecture

- Single global `AudioContext`, created on first user interaction (browser autoplay policy)
- **Typing sounds**: Web Audio API real-time synthesis. Short-lived OscillatorNode/BufferSource per keystroke, auto-released after playback
- **Background ambiance**: Base64-encoded short audio clips decoded to AudioBuffer, played with `loop: true`
- Two independent GainNodes for separate volume control (typing vs background)

## Theme System

4 visual themes, each defines background, text color, accent color, toolbar style:

| Theme | Background | Text | Mood |
|-------|-----------|------|------|
| Deep Night | Dark blue-black gradient | Warm white | Quiet, immersive |
| Sunset | Deep purple-orange gradient | Light gold | Warm, romantic |
| Dawn | Light cream white | Dark gray | Fresh, bright |
| Ocean | Deep cyan-blue gradient | Pale cyan-white | Calm, open |

Theme-music pairing (default recommendation, user can freely mix):
- Deep Night → Rainy Night
- Sunset → Fireplace
- Dawn → Forest
- Ocean → Silence

Themes implemented via CSS custom properties (CSS Variables) for instant switching.

## Markdown Export

- Toolbar export button generates `.md` file and triggers browser download
- Filename: first line of text as title (e.g., `My Article.md`), or `写作-YYYY-MM-DD.md` if first line is empty

## Data Persistence (localStorage)

- Auto-save with 1-second debounce after each keystroke
- Stores: text content, cursor position, current theme, music selection, typewriter style, volume settings
- Full state restoration on page reload
- Toolbar shows brief "saved" indicator after each save

## New Document

- Toolbar "New" button
- Confirmation dialog if current content is not empty
- Clears editor for fresh start

## Editor Implementation

- `contenteditable` `<div>` (not `<textarea>`) for flexible cursor and character styling
- Typewriter animations via CSS animation classes, added per character, removed after animation completes
- `requestAnimationFrame` for animations, non-blocking input

## Responsive Design

- Desktop: centered writing area, max-width 700px
- Mobile: full-width writing area, toolbar becomes bottom sheet
- Primary target is desktop; mobile should be usable but not optimized

## Browser Compatibility

- Target: Chrome 80+, Safari 14+, Firefox 78+, Edge 80+
- No IE support

## File Structure

Single `index.html` containing all HTML, CSS, and JavaScript. Base64 audio clips embedded as data URIs within the JavaScript section.

CNAME file retained for custom domain configuration.
