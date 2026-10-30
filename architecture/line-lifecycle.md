# Line lifecycle

One spoken line moves through fixed stages. Synthesis of the next line may overlap playback of the current one, bounded by the active engine’s **synth ahead** setting.

## Stages

| Stage | Thread | What happens |
| --- | --- | --- |
| Submit | UI | Slash commands parsed first (if any). Plain text is refused while the speaking engine is not ready. Remaining text is scrubbed for markers. Engine, model, voice, styles, pace, speech language, pitch, translation choices (and preset label) are **frozen** on the line. Optional “split sentences” enqueues one entry per sentence. |
| Queue | — | Line waits as `queued`. |
| Synthesize | Synth worker | Acquire ahead permit; take oldest line → `synthesizing`. |
| Translate | Synth worker | If translation is on, translate text runs into the line's speech language (else the voice's locale); markers pass through. Failure speaks the original with a note. See [Translation](translation.md). |
| Cache / network | Synth worker | Expand `[date]` / `[time]` and `time:` parts from local now (speech language, else the engine's default locale, else the voice's locale, else English) **before** fingerprint and adapter input. The expanded text is the line's spoken text; mouth analysis, the playback feed, and captions reuse it. `[time]` is hour and minute (same-minute cache hits); `[time:full]` includes seconds; `[time:24h]` is `HH:MM`; `[time:tod]` is the day period. If cache-read is allowed and the fingerprint hits, load cached audio. Otherwise call the adapter. Cache-write (when enabled) stores pre-pitch audio. Then apply pitch → `ready`. |
| Drop or play | Synth / Play | If the session was cleared, drop the clip. Else hand off to the play queue. |
| Play | Play worker | Release ahead permit; optional mouth analysis; captions / avatar follow; play clip → `playing`. |
| Done | Play worker | Clip ends or stop; avatar zeros; captions clear after a short idle delay → terminal state. |

Adapter work and disk/network cache I/O never run on the Qt thread.

## Frozen capture

While a line is in flight, live chip changes do **not** rewrite that line. Replay and regenerate reuse the frozen capture on the line’s status object. Live submits snapshot current settings at submit time.

## Speech cache (design)

- Fingerprint ignores temperature and pitch; pitch is applied after load.
- Fingerprint uses the spoken (translated, expanded) text, so each target language caches its own audio.
- An adapter may add a cache version to its fingerprint (Kokoro bumps it when its phonemes change), so stale takes miss without re-keying other engines' caches.
- Per-engine toggles for write, prefer-read, and size budget.
- Cloud engines may meter usage and define over-limit behavior (continue synthesizing, stop, or play from cache only).
- Cache hits for metered engines record savings without counting as new billed synthesis.

## Clear and stop

Clearing the session stops playback and drops queued / ready work without paying for unbounded ahead synthesis. Stop targets the current play path; module and Speak ownership of lines is tracked via a line **source** so stop-mine semantics stay well-defined.
