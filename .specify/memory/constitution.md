# Transpose Chords — Constitution

## Inheritance

Slopsmith's core plugin contract governs everything in this repo (manifest,
the `/ws/highway/` WebSocket protocol, the `highway` global). This
constitution lists Transpose Chords' own non-negotiables.

## Core Principles

### I. WebSocket-proxy interception is opt-in and local
The plugin patches `window.WebSocket` once
(`window.__transposeChordsSocketPatched`) to observe `/ws/highway/`
messages and rewrite `chord_templates`. The patch MUST be installed only
when the user enables the toggle and MUST be idempotent. It MUST forward
all other messages and traffic verbatim — this is a tap, not a filter.

### II. Tuning detection from `song_info`
Semitone offset is computed from the `song_info` payload's `tuning`
array (assumed uniform across strings; if not uniform, semitones=0). The
code uses `low-string-first` semantics and inverts the sign — a `-2`
tuning offset (D Standard) becomes `+2` semitones to transpose chord
names back to E Standard.

### III. Reversibility
Every chord template the plugin rewrites MUST be remembered in
`originalChordNames` keyed by template id. Toggling off restores every
remembered name. State MUST NOT leak between songs — `originalChordNames`
is cleared on disable.

### IV. No backend, no persistence
The plugin is purely client-side. Toggle state is ephemeral; no
`localStorage`, no API endpoint. Constitution §III implies that the next
load starts disabled.

### V. Sharps preferred, flats preserved
When transposing, if the original root was flat (`Bb`, `Db`, …), the
transposed root keeps the flat representation; otherwise sharps are used.
Avoids surprises for users who think in flats.

## Governance

The supported tuning range and the semitone math are unit-testable; any
change to `transposeChord` or `computeSemitonesFromTuning` should ship
with a synthetic test fixture (none exists today; tracked in `tasks.md`).

**Version**: 0.1.0 | **Ratified**: 2026-05-10 | **Last Amended**: 2026-05-10
