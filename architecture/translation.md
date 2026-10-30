# Translation

MiraSpeak can translate a line before speaking it. Translation is built in (not a module), off by default, and runs on the synth worker, never on the UI thread.

## Settings

| Setting | Values |
| --- | --- |
| Translate | On / Off (a global toggle; the **Translate** chip after Pitch on Speak and the quick bar, or `/translate`) |
| Provider | **Azure Translator** (cloud, REST v3) or **Argos** (local CTranslate2 + SentencePiece models) |
| From | **Auto** (local language detection) or a fixed source language |

There is no separate target setting. The target is the line's **speech language**, else the voice's own locale. A voice with neither isn't translated. Changing the speech language (chip or `/speech`) changes the target.

These choices are frozen on the line at submit with the rest of the voice capture, so changing them later doesn't affect lines already in flight. Replay and Regen reuse the line's frozen choices.

## Per line

1. Split the line into text runs and markers. Bracket markers and `[[` escapes pass through untouched; only text runs go to the provider.
2. Resolve the source. With **Auto**, detect one language for the whole line. If detection is unsure, or the source matches the target, the line is spoken as typed.
3. Look up each run in the translation cache; send only the misses to the provider.
4. Rejoin the runs and markers. That text is what the engine speaks; then `[date]` / `[time]` expand (see [Line lifecycle](line-lifecycle.md)).

If translation fails (no key, no installed pair, network error, bad output), the original text is spoken and the line carries a translation note. The line itself does not fail.

## Providers

- **Azure Translator** — key in the OS credential store, separate from the Azure Speech key. Region is optional. Characters served from cache count toward a saved-characters meter.
- **Argos** — language pairs download into local app data. A missing direct pair pivots through English, and downloading it installs both legs. Decoding uses a repetition penalty and a length cap so small models don't loop; output still stuck on one repeated character is treated as a failure (spoken as typed, never cached).

## Caching

- One SQLite translation cache, keyed by provider, model version, source, target, and text run. It is LRU by last use with a fixed size cap, can be turned off, and has a Clear button on the Translation page.
- Argos model versions include a decode version, so changing decoding settings misses old entries instead of reusing them.
- The speech cache fingerprints the **translated** text, so each target language caches its own audio.
- The translation cache and the speech cache share one canonical hash (sorted-key JSON → SHA-256).

UI surfaces that need to know a line's text without speaking it (cache marks, fingerprints) only peek at the cache and never call a provider. The Translation page's **Try** row previews a translation in the background without speaking.

## Surfaces

- **Settings:** a Translation nav group with Translation (toggle, provider, source, cache, Try), Azure Translator (key, region), and Argos (download, installed pairs).
- **Speak:** a translated row shows the translation alongside the original.
- **Captions:** Text Output can write the translated text, the original, or both.
- **MCP:** `set_translation` (on/off, provider, source), `get_status` includes translation state, and `list_translation_languages`. There are no MCP tools for keys, Argos downloads, or clearing the cache.
