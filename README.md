# Shulker Ambient Player

A single-file, offline-capable web player that reproduces Minecraft's
random ambient-sound mechanic for the **Shulker** mob. All audio samples are
embedded as base64 data URIs, so the page works with no network, no build
step, and no external dependencies — just open `shulker_ambient_player.html`.

---

## Features

- **7 embedded Shulker ambient samples** (OGG / MP3), bundled in the HTML file.
- **Random scheduler** — plays one sound on start, then waits a jittered
  interval (±40%) before the next one, mimicking vanilla Minecraft's ambience.
- **Two playback modes**
  - *Scheduled* — continuous random ambience until stopped.
  - *Manual* — fire a single random sound, or click any track in the list to
    play that specific sound.
- **6 interface languages** — 中文 / Español / English / العربية / Русский /
  Français, selectable from a dropdown, auto-detected from the browser on first
  visit, and persisted in `localStorage`.
- **Live controls** — volume slider and interval slider take effect immediately.
- **Status HUD** — shows state, last-played track + timestamp, play counter, and
  a progress bar for the current sound.
- **Keyboard shortcut** — <kbd>Space</kbd> toggles start/stop.
- **Pixel-art UI** — Minecraft-adjacent panel styling, no images required.

---

## Quick start

1. Download `shulker_ambient_player.html`.
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari).
3. Press **▶ Start**.

No server, no install, no internet connection required.

---

## Usage

### Buttons

| Control | Behavior |
| --- | --- |
| **▶ Start** | Immediately plays one sound, then continues with random intervals. |
| **✦ Play one** | Plays a single random sound. Does not start the scheduler. |
| **■ Stop** | Stops playback, cancels pending timers, and clears the track highlight. |

### Sound list

Each row in the **Sounds (7)** list is clickable:

- Clicking a row plays **that specific sound**, with identical behavior to the
  *Play one* button (single play, no scheduler side-effects).
- While any sound is currently playing, clicks are ignored (same mutex rule as
  the manual button).
- If the scheduler is running, the random chain resumes automatically after the
  clicked sound finishes.
- The currently playing row is highlighted in amber.

### Sliders

- **Volume** — 0–100%, applied live to the current and future sounds.
- **Interval** — base interval in seconds (2–30). The actual delay is the base
  value multiplied by a random factor in `[0.6, 1.4]`, i.e. ±40% jitter.

### Keyboard

- <kbd>Space</kbd> — toggle between Start and Stop.

---

## Behavior model

The player uses a simple three-state machine:

```
stopped ──[Start]──▶ playing ──[ended]──▶ idle ──[timer]──▶ playing ──▶ …
   ▲                    │
   └────[Stop]──────────┘
```

- **stopped** — idle, no timers, no highlight.
- **playing** — audio is actively playing (`isPlaying === true`).
- **idle** — scheduler waiting for the next random interval; a countdown
  `Idle, next in Xs` is shown in the HUD.

Rules:

- A **global mutex** (`isPlaying`) prevents overlapping sounds regardless of
  trigger source (scheduler, *Play one*, or list click).
- A manual trigger or list click while the scheduler is running **cancels the
  pending timer** and plays immediately; on `ended`, the scheduler re-arms.
- A manual trigger or list click while stopped plays once and returns to
  **stopped**.
- `pick()` avoids immediately repeating the previous track with 80% probability
  (only when more than one sound is available).

---

## Internationalization

Six locales are bundled in the `I18N` object:

| Code | Language |
| --- | --- |
| `zh` | 中文 |
| `es` | Español |
| `en` | English |
| `ar` | العربية |
| `ru` | Русский |
| `fr` | Français |

- On first load, the language is chosen from `navigator.language` (matching the
  primary subtag against the six codes above); the fallback is `en`.
- The selection is saved to `localStorage` under `shulker_lang`.
- The dropdown in the top-right corner switches languages instantly; the
  document `<title>` and all `[data-i18n]` nodes are re-rendered, and any
  dynamic HUD text is rebuilt in the new language.

> **Note:** Arabic content is provided as translated strings, but the panel
> layout is not switched to RTL. To enable RTL, add
> `document.documentElement.dir = (lang === "ar") ? "rtl" : "ltr";` inside
> `applyLang()`.

---

## Technical notes

- **Single file.** All CSS, JS, and audio are inlined. There are no external
  requests, fonts, or scripts.
- **Audio format.** Samples are stored as `data:audio/ogg;base64,...` (and one
  `data:audio/mpeg`), decoded by the browser via the standard `Audio` element.
  Safari and Firefox both support OGG Vorbis; the MP3 sample covers legacy
  fallback.
- **No build tooling.** Plain ES5-compatible JavaScript, no frameworks.
- **File size.** The embedded audio dominates the file size; it will be roughly
  several hundred kilobytes depending on the source samples.

### Browser support

Tested against current Chrome, Edge, Firefox, and Safari. Any browser that
supports `<audio>`, `localStorage`, and `requestAnimationFrame` will work.

---

## Customization

- **Add or replace sounds** — edit the `SOUNDS` array; each entry is
  `{ name, data }` where `data` is a data URI. Update the `sounds` translation
  key (`"Sounds (7)"`) and the initial `<h3 id="soundsTitle">` if the count
  changes.
- **Change interval bounds** — adjust `min`/`max` on the `#gap` range input and
  the jitter formula in `schedule()` (`0.6 + Math.random() * 0.8`).
- **Change volume default** — set `value` on `#vol` (currently `70`).
- **Add a locale** — add a new object to `I18N`, extend `SUPPORTED`, and add an
  `<option>` to `#langSelect`.

---

## Changelog

Relative to the original version:

- **English comments** — all inline comments translated from Chinese to English.
- **Six locales** — i18n expanded from `zh`/`en` to `zh`, `es`, `en`, `ar`,
  `ru`, `fr`.
- **Language dropdown** — the `EN`/`中文` toggle button was replaced by a
  `<select>` with one option per language; auto-detection and persistence
  updated accordingly.
- **Clickable track list** — clicking any row in the Sounds list plays that
  specific sound, using the same mutex and timer-cancellation rules as the
  *Play one* button. Hover styling and a pointer cursor were added.
- **`playOne(manual, forcedIndex)`** — optional second argument selects a
  specific track instead of a random one; existing call sites are unchanged.

---

## Credits & license

- Audio samples originate from **Minecraft** (© Mojang Studios). They are
  included here for personal, non-commercial, educational use only. Do not
  redistribute the embedded audio commercially.
- The player code is provided as-is for learning and tinkering.
