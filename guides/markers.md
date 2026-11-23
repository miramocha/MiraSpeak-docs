# Marker tokens

Accidental `[brackets]` in the line are stripped and never spoken. `[[` is a literal `[`.

`[date]` and `[time]` are read out as the current date and time when the line is spoken. These
tokens request a part of the time: `[time:full]`, `[time:24h]`, `[time:tod]`, `[time:h]`,
`[time:m]`, `[time:s]`. The stored line keeps the tokens, so Replay says the new time.

Your own marker shortcuts live under **Voice → Marker aliases** (Import / Export as JSON).
