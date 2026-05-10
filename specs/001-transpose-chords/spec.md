# Spec — Transpose Chords (`transpose-chords`)

> Retrospective spec for shipped v0.1.0. Implementation is the single file
> `screen.js` (178 lines).

## Summary

A Slopsmith plugin that rewrites chord names displayed on the highway so a
song in a non-standard tuning reads as if it were in E Standard. For
example, in Eb Standard tuning a chord named `D#7` is rewritten to `E7`.
Activated via a toggle button in the player controls.

## User stories

### US-1 — Toggle Transpose Chords
- **Given** I'm on the player screen,
  **When** I click the **Transpose Chords** button,
  **Then** the plugin enables, installs a one-time WebSocket proxy
  (`window.__transposeChordsSocketPatched`), and immediately transposes
  any already-loaded chord templates if `semitones !== 0`.

### US-2 — Detect tuning from song_info
- **Given** a song loads and the highway WebSocket emits a `song_info`
  message,
  **When** the proxy intercepts it,
  **Then** `semitones = computeSemitonesFromTuning(msg.tuning)`.
  Algorithm:
  - Empty / non-array tuning → 0.
  - Tuning offsets uniform → `-tuning[0]` (low-string offset, sign-inverted).
  - Non-uniform tunings → 0 (out of scope).

### US-3 — Rewrite chord templates in flight
- **Given** the plugin is active and `semitones !== 0`,
  **When** the proxy intercepts a `chord_templates` message,
  **Then** it transposes every `tmpl.name` and stashes the original in
  `originalChordNames.get(tmpl.id)`. The rewritten message is forwarded
  to the original `onmessage` handler.

### US-4 — Toggle off restores chord names
- **Given** transposed templates are visible,
  **When** the user clicks the toggle to disable,
  **Then** the plugin walks `window.highway.chordTemplates`, restores
  each name from `originalChordNames`, and clears the map.

### US-5 — Apply to current song on enable
- **Given** the plugin is enabled mid-song with `semitones !== 0`,
  **When** the toggle is clicked,
  **Then** existing `window.highway.chordTemplates` entries are
  transposed in place using the cached `semitones`.

### US-6 — Note name preference
- **Given** a chord whose root is `Bb`,
  **When** transposed by +1 semitone,
  **Then** the output uses flat notation (`B`).
- **Given** a chord whose root is `D#`,
  **When** transposed by +1 semitone,
  **Then** the output uses sharp notation (`E`).

## Functional requirements

| ID    | Requirement                                                                                          | Source       |
|-------|------------------------------------------------------------------------------------------------------|--------------|
| FR-1  | Inject a **Transpose Chords** button into `#player-controls`, with on/off styling.                   | `screen.js`  |
| FR-2  | Patch `window.WebSocket` once (`__transposeChordsSocketPatched` flag).                                | `screen.js`  |
| FR-3  | The proxy class MUST forward `onopen`, `onclose`, `onerror`, `addEventListener`, etc., transparently. | `screen.js`  |
| FR-4  | Compute semitone offset from `song_info.tuning` (uniform offsets only; non-uniform → 0).             | `screen.js`  |
| FR-5  | Transpose chord names via `transposeChord(chord, semitones)` with sharp/flat preference preserved.    | `screen.js`  |
| FR-6  | Remember pre-transpose names in `originalChordNames` keyed by template id.                            | `screen.js`  |
| FR-7  | On toggle off, restore every remembered name and clear the map.                                       | `screen.js`  |
| FR-8  | Suffix preservation: only the chord root is transposed; everything after the root (e.g. `7`, `m`, `sus4`) is left intact. | `screen.js`  |
| FR-9  | Auto-trigger initial state on injection: the button's first `click()` is dispatched programmatically to apply current state to a loaded song. | `screen.js` `injectBtn` |

## Non-functional

- **Latency**: each `chord_templates` message is processed synchronously
  inside `onmessage`; overhead is bounded by chord count (typically <50).
- **No telemetry, no persistence**.
- **Browser**: depends on `window.WebSocket` constructor being patchable
  pre-WebSocket-creation by other plugins.

## Out of scope

- Non-uniform tunings (e.g. drop tunings where one string is offset
  differently). Returns 0 today.
- Modifying note pitches or any non-chord-name display.
- Persisting the toggle state.
- Tunings beyond standard semitone offsets (e.g. quarter-tone tunings).

## Open clarifications

- [NEEDS CLARIFICATION] How does this interact with sloppak / sloppak
  converter outputs that may have already been re-tuned to E Standard?
  The README mentions "song was not converted" — what marker indicates a
  converted song?
- [NEEDS CLARIFICATION] Should drop tunings (one string offset) be
  supported by transposing only the affected string? Today the plugin
  refuses (semitones=0).
- [NEEDS CLARIFICATION] What is the tuning offset sign convention in
  `song_info`? The plugin assumes "negative = down from E Standard"; if
  this changes the entire transposition flips sign.
- [NEEDS CLARIFICATION] Plugin id uses `transpose-chords` (kebab-case)
  while most plugins use snake_case ids. Stay consistent or rename?
