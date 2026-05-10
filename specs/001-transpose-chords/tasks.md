# Tasks — Transpose Chords

Status legend: `DONE` (shipped in v0.1.0), `OPEN` (not yet implemented), `[P]` (parallelisable).

## US-1 — Toggle button
- [DONE] Inject `#btn-transpose-chords` into `#player-controls`.
- [DONE] On/off styling.
- [DONE] Idempotent injection (id check).
- [DONE] Auto-click after injection to apply default state.

## US-2 — Tuning detection
- [DONE] `computeSemitonesFromTuning(tuning)`.
- [DONE] Uniform-tuning gate (non-uniform → 0).

## US-3 — Live rewrite
- [DONE] Patch `window.WebSocket` once.
- [DONE] Forward all non-highway sockets verbatim.
- [DONE] Intercept `chord_templates` messages and rewrite names.
- [DONE] Re-emit synthetic `MessageEvent` with rewritten JSON.

## US-4 — Restore on disable
- [DONE] Walk `window.highway.chordTemplates` and restore names.
- [DONE] Clear `originalChordNames` map.

## US-5 — Apply to current song on enable
- [DONE] Iterate `chordTemplates` in `toggle()` when enabling.

## US-6 — Sharp / flat preservation
- [DONE] Decision based on whether original root contained `b`.

## Cross-cutting
- [DONE] Idempotent proxy install (`__transposeChordsSocketPatched`).
- [DONE] Console logging for transposition events.
- [OPEN] [P] Drop-tuning support: transpose only the affected string's chord names (Q5).
- [OPEN] [P] Read a "song was already converted" marker (Q in spec).
- [OPEN] [P] Plugin id naming alignment with `transpose_chords` (Q7).
- [OPEN] Unit tests for `transposeChord` and `computeSemitonesFromTuning`.
- [OPEN] [P] Persist toggle state across loads (today: ephemeral; constitution §IV).
- [OPEN] Investigate non-Latin / unicode chord names (e.g. `B♭`) — currently passed through.
