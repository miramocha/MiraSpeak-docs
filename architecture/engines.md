# Voice engines

MiraSpeak speaks through **adapters**. The UI never embeds provider-specific style or voice tables; it reads `Capabilities` from the active adapter (and model, where the provider has one).

## Adapter contract (conceptual)

Each adapter exposes:

- Whether it's local or cloud, and whether it's currently configured  
- A default speech locale (or the voice's own locale), used for marker expansion and translation  
- Models (if any), voices, styles, paces, speech languages  
- Inline marker catalog (what `[id]` tokens survive scrubbing); the registry always adds MiraSpeak `[date]` / `[time]` on top (`[time:full]` / `[time:24h]` / `[time:tod]` / `[time:h|m|s]` share the time keep id)  
- `multi_style`: checklist Style picker vs single-select  
- `synthesize(request) →` audio clip  

Unsupported styles selected in the UI degrade to that engine’s default.

## Built-in engines

| Engine | Group | Notes |
| --- | --- | --- |
| Kokoro | In-app (ONNX) | Offline synthesis; styles/paces from adapter capabilities. Each voice speaks its own language (see below). |
| Windows | In-app, Windows only | The voices already installed in Windows. |
| VOICEVOX | Local server (HTTP) | Expects a running VOICEVOX engine. Character + style (mood) selection; catalog labels are user-facing. |
| Local TTS | Local server (HTTP) | A TTS server on localhost (default `http://127.0.0.1:8020`). The address must be `localhost`, `127.0.0.1`, or `[::1]`. |
| Azure Speech | Cloud | One Azure engine. The Engine chip shows the model: DragonHD, Omni, or Neural. Multi-style checklist where the model supports it. Secrets in keyring. Optional usage meter; **At limit** is Continue, Hard stop, or Play from cache if available. Setup: [Azure guide](../guides/azure-speech-translator.md). |

Engines share the same submit → synth → play path. Generate ahead and cache policies are configured per engine id (Azure models share one Azure bucket).

## Engine modules

The registry starts with the built-in engines. A module with the `voice_engine` permission can add more while it runs (see [Modules](modules.md#engine-modules)).

| Engine | Group | Notes |
| --- | --- | --- |
| Piper | In-app (ONNX, CPU) | Official Piper module. English Piper voices, one ONNX file per speaker, downloaded one voice at a time. See the [Piper guide](../guides/piper.md). |

- **Lifetime:** a module engine is added once its module is running and dropped when it stops. Its settings table stays, so the engine comes back as it was.
- **Leases:** each line holds its adapter from start to finish. Dropping or resetting an engine mid-line closes the adapter only after that line is done.
- **Fallback:** a line whose engine isn't loaded (its module isn't installed, enabled, or started yet) speaks with Kokoro's first voice, default style, and default pace. The fallback happens when the line is submitted, so the speech cache stores it under Kokoro. Settings don't change: the persona, preset, or live voice still names the module engine, and uses it again once the module runs. The line's row says why, for example "piper is not installed or enabled; using Kokoro".

## Kokoro languages

A Kokoro voice id's first letter is its language (`a` en-US, `b` en-GB, `e` es-ES, `f` fr-FR, `h` hi-IN, `i` it-IT, `j` ja-JP, `p` pt-BR, `z` zh-CN). That letter drives the voice banks, the speech locale (and so the [translation](translation.md) target), and the grapheme-to-phoneme language. MiraSpeak phonemizes on the synth worker and passes phonemes to Kokoro:

- Japanese uses OpenJTalk readings mapped through misaki's kana → IPA table (the G2P Kokoro was trained on), not espeak, which spells words out on small kana and some punctuation.
- The other languages use espeak-ng with language-switch flags removed. English phonemes stay identical to kokoro-onnx's own.

When Kokoro's phoneme output changes, its cache version bumps so old takes miss once; other engines' caches are untouched.

## Readiness checks

Each engine is checked in the background (a module engine from its adapter's own readiness check): at launch, when its settings page opens, after Test / Refresh / key / URL / download changes, on engine or preset switch, and after a failed line. VOICEVOX is re-polled while it's failing and in view or in use; Azure is never polled.

- You can still switch to an engine that isn't ready (picker, preset, MCP). While the **speaking** engine isn't ready, speech is gated: the Speak button and module controls that speak are off, and Enter on plain text is refused. Slash commands still run so you can switch away, except `/render`, which needs the engine. Replay and Regen stay on (frozen engine, may hit cache).
- The voice bar footer shows the failing engine's summary with **Check again** and **Open settings**; the nav marks in-use failing engines with a dot. Nothing navigates to a settings page on its own.
- Provider pages always show connection and local rows (cache, Generate ahead, usage); only the voice/character catalog waits behind the check.
- MCP, hotkeys, and Live chat aren't gated.

## Engine-agnostic UI checklist

- Style and pace chips bind to `Capabilities`, not hardcoded lists.  
- Switching engine or model coerces multi-style selections down to one style when the new target isn't multi-style.  
- Marker pickers and alias targets come from the union of adapter catalogs, labeled by kind and source.  
- Provider-specific pages (Test / Refresh / cache / usage) live under Voice engines, on In-app, Local server, or Cloud; Speak itself stays catalog-driven.
