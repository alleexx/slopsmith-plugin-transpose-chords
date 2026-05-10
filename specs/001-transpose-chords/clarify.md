# Clarifications — Transpose Chords

## Q1 — Why patch `window.WebSocket` instead of hooking the highway?
**Resolved.** Slopsmith doesn't expose a public hook for "rewrite a
highway message before the highway sees it". Patching the WebSocket
constructor at module load is the only place we can intercept
`chord_templates` before render. The patch forwards everything verbatim
otherwise (constitution §I).

## Q2 — Why does the URL-suffix check (`/ws/highway/`) gate the
interception?
**Resolved.** Other plugins or core endpoints may use WebSockets too. Only
highway sockets carry chord templates. Restricting interception to the
highway URL avoids inadvertently corrupting other channels.

## Q3 — How is sharp/flat preference inferred?
**Resolved.** From the original chord root: `Bb` and similar contain `b`,
so the rewrite stays in flat space; everything else uses sharps. This
preserves the original notation style.

## Q4 — Why is the toggle disabled by default?
**Resolved.** The README says "active by default when installed" and the
code calls `button.click()` after injection. So the plugin actually
auto-enables on injection; we then toggle off would be a manual click.
[Note] This contradicts §IV "next load starts disabled" — the auto-click
re-enables it. Captured here as a known divergence; see `analyze.md`.

## Q5 — What does the README mean by "if … song was not converted"?
**Open.** The README implies there's a marker in the manifest indicating
a song has already been transposed (e.g. by `sloppak-converter`'s tuning
normalisation). The current code doesn't read any such marker — it
unconditionally transposes when `semitones !== 0`.

## Q6 — Does the plugin handle every chord name shape?
**Resolved.** The regex `/^([A-G][#b]?)(.*)$/` matches a root of one
letter optionally followed by `#`/`b`, then any suffix. Chords starting
with a slash (`/Bb`) or unusual notation (e.g. `B♭`) are NOT matched and
pass through untouched.

## Q7 — Why is the plugin id `transpose-chords` rather than
`transpose_chords`?
**Open.** Inconsistent with most Slopsmith plugins (which use snake_case
ids). Renaming would be a breaking change for anyone keying on the
manifest id.

## Q8 — How do the WebSocket proxy's static getters relate to the native
constructor?
**Resolved.** `CONNECTING`, `OPEN`, `CLOSING`, `CLOSED` static getters on
the proxy class delegate to `NativeWebSocket.*`. This keeps libraries
that compare `ws.readyState === WebSocket.OPEN` working when they import
the proxy.
