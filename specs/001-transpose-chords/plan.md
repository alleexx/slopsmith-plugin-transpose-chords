# Plan — Transpose Chords (as built)

## File map

| File          | Lines | Purpose                                                         |
|---------------|-------|-----------------------------------------------------------------|
| `plugin.json` | 5     | Manifest. `id: transpose-chords`, version `0.1.0`, declares `screen.js`. |
| `screen.js`   | 178   | All logic: button injection, WebSocket proxy, `transposeChord`, `computeSemitonesFromTuning`, restore-on-disable. |
| `README.md`   | 17    | Short user docs.                                                |

## Architecture

```
                    window.WebSocket  (native)
                            │
                  patched once per page
                            │
                            ▼
                  TransposeWebSocket class
                  (forwards open/close/error/etc.)
                            │
                  on message:
                            │
                  is URL '/ws/highway/'?
                       │           │
                      yes          no  ──► forward ev
                       │
                       ▼
                  parse JSON
                       │
                  ┌────┴────────────────────────────────┐
                  │                                     │
                msg.type === 'song_info'                msg.type === 'chord_templates' ?
                  │                                     │
                  ▼                                     yes (and active and semitones)
        semitones = computeSemitonesFromTuning(msg.tuning)
                                                        │
                                                        ▼
                                              for each tmpl in msg.data:
                                                originalChordNames.set(id, name)
                                                tmpl.name = transposeChord(name, semitones)
                                              re-emit synthetic MessageEvent with new JSON
```

## Helpers

```js
function transposeChord(chord, semitones) {
    if (!chord || !semitones) return chord;
    const sharp = ['C','C#','D','D#','E','F','F#','G','G#','A','A#','B'];
    const flat  = ['C','Db','D','Eb','E','F','Gb','G','Ab','A','Bb','B'];
    const m = chord.match(/^([A-G][#b]?)(.*)$/);
    if (!m) return chord;
    const root = m[1];
    const isFlat = root.includes('b');
    const idx = sharp.indexOf(/* normalised root */);
    return (isFlat ? flat : sharp)[(idx + semitones + 12) % 12] + m[2];
}

function computeSemitonesFromTuning(tuning) {
    if (!Array.isArray(tuning) || tuning.length === 0) return 0;
    if (!tuning.every(v => v === tuning[0])) return 0;  // non-uniform → 0
    return -tuning[0];
}
```

## Toggle behaviour

```js
function toggle() {
    active = !active;
    if (active) {
        createWebSocketProxy();
        // Apply to current chord templates already on highway
        for (const tmpl of window.highway.chordTemplates || []) {
            if (tmpl.name && !originalChordNames.has(tmpl.id) && semitones !== 0) {
                originalChordNames.set(tmpl.id, tmpl.name);
                tmpl.name = transposeChord(tmpl.name, semitones);
            }
        }
    } else {
        // Restore + clear
        for (const tmpl of window.highway.chordTemplates || []) {
            if (originalChordNames.has(tmpl.id)) tmpl.name = originalChordNames.get(tmpl.id);
        }
        originalChordNames.clear();
    }
}
```

## Idempotency

- `window.__transposeChordsSocketPatched` ensures one proxy install per
  page.
- The button's `id="btn-transpose-chords"` is checked before re-injecting.
- The injection ends with `button.click()` to auto-apply the current state
  to a loaded song; this is the README's "active by default" behaviour.

## Risks / drift watchpoints

- **`song_info.tuning` shape** — assumed array of integer semitone
  offsets. Any other shape returns 0.
- **`chord_templates` payload** — assumed `{type, data: [{id, name, …}]}`.
- **`window.highway.chordTemplates`** — read on toggle for in-flight
  rewrites; if the highway later moves chord state, this breaks.
- **Native WebSocket invariants** — the proxy class re-implements every
  surface used by callers. New WebSocket APIs (e.g. `send` with
  Streaming) would not flow through.
- **Drop tunings** — not supported.
