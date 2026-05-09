# Analyze — Transpose Chords

## Coverage

| Area              | Spec | Plan | Code         | Notes                                  |
|-------------------|------|------|--------------|----------------------------------------|
| Toggle UI         | ✅   | ✅   | `screen.js`  | One button, auto-clicked on injection  |
| WebSocket proxy   | ✅   | ✅   | `screen.js`  | Patched once, forwards verbatim        |
| Tuning detection  | ✅   | ✅   | `screen.js`  | Uniform-tuning gate                    |
| Chord rewrite     | ✅   | ✅   | `screen.js`  | Sharp / flat preserved                 |
| Restore on toggle | ✅   | ✅   | `screen.js`  | `originalChordNames` map               |
| Tests             | ❌   | ❌   | —            | None                                   |

## Drift

- **Constitution §IV vs auto-click**: §IV says "next load starts disabled".
  The README and `injectBtn` say "active by default". Since the
  `button.click()` line auto-toggles to active immediately after
  injection, the runtime behaviour is "starts disabled, instantly
  enables" — effectively active. Update either §IV or remove the
  auto-click depending on intent.
- README notes "Currently supports standard tunings from C Standard to B
  Standard"; the code transposes any uniform offset so technically
  arbitrary uniform offsets work. Match the docs to the code.
- Plugin id `transpose-chords` (kebab-case) vs ecosystem convention of
  snake_case. Cosmetic but inconsistent (Q7).

## Gaps

1. **Drop tunings unsupported** (Q5). Returns semitones=0 silently.
2. **Conversion marker not honoured** (Q in spec) — songs already
   normalised to E Standard would be transposed twice.
3. **Toggle state ephemeral** — every page load starts disabled (or
   active-via-auto-click).
4. **No tests** — pure functions ripe for testing.
5. **Unicode chord names** (`B♭` vs `Bb`) pass through untouched.
6. **Slash chord roots** (`C/Bb`) match the regex root as `C` and the
   slash bass is left alone — depending on user expectation, this may be
   the "wrong" half of the chord rewriting.

## Recommendations

- **Resolve §IV vs auto-click.** Either remove the auto-click and let the
  user choose, or remove §IV. The README should match.
- **Add a "song already converted" guard** by reading a marker in
  `song_info` (e.g. `tuning_normalised: true`) and short-circuiting.
- **Support drop tunings** by exposing a per-string offset path:
  `chord_templates` carry per-string fingerings and could be transposed
  selectively.
- **Add unit tests** for `transposeChord` (sharp + flat round-trips,
  octave wrap, unknown roots) and `computeSemitonesFromTuning`
  (uniform / non-uniform / empty / null).
- **Slash chords**: extend the regex to capture `^([A-G][#b]?)([^/]*)(/[A-G][#b]?)?$`
  and transpose both root and bass independently.
- **Snake-case id**: rename to `transpose_chords` aligned with the rest
  of the ecosystem; cut a major version (`1.0.0`) for the rename.
