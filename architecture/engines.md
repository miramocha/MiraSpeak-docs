# Voice engines

MiraSpeak speaks through **adapters**. The UI never embeds provider-specific style or voice tables; it reads `Capabilities` from the active adapter (and model, where the provider has one).

## Adapter contract (conceptual)

Each adapter exposes:

- Whether it is local or cloud, and whether it is currently configured  
- A default speech locale (or the voice's own locale), used for marker expansion and translation  
- Models (if any), voices, styles, paces, speech languages  
- Inline marker catalog (what `[id]` tokens survive scrubbing); the registry always adds MiraSpeak `[date]` / `[time]` on top (`[time:full]` / `[time:24h]` / `[time:tod]` / `[time:h|m|s]` share the time keep id)  
- `multi_style` — checklist Style picker vs single-select  
- `synthesize(request) →` audio clip  

Unsupported styles selected in the UI degrade to that engine’s default.

## Shipped engines

| Engine | Where | Notes |
| --- | --- | --- |
| **Kokoro** | Local (ONNX) | Offline synthesis; styles/paces from adapter capabilities. Each voice speaks its own language (see below). |
| **VOICEVOX** | Local HTTP | Expects a running VOICEVOX engine. Character + style (mood) selection; catalog labels are user-facing. |
| **Azure Speech** | Cloud | One Azure settings surface with a **model** chip (e.g. Dragon HD / Omni / Neural). Multi-style checklist where the model supports it. Secrets in keyring. Optional usage meter and limit action. Setup: [Azure guide](../guides/azure-speech-translator.md). |

Engines share the same submit → synth → play path. Synth-ahead and cache policies are configured per engine id (Azure models share one Azure bucket).

## Kokoro languages

A Kokoro voice id's first letter is its language (`a` en-US, `b` en-GB, `e` es-ES, `f` fr-FR, `h` hi-IN, `i` it-IT, `j` ja-JP, `p` pt-BR, `z` zh-CN). That letter drives the voice banks, the speech locale (and so the [translation](translation.md) target), and the grapheme-to-phoneme language. MiraSpeak phonemizes on the synth worker and passes phonemes to Kokoro:

- **Japanese** uses OpenJTalk readings mapped through misaki's kana → IPA table (the G2P Kokoro was trained on), not espeak, which spells words out on small kana and some punctuation.
- **Other languages** use espeak-ng with language-switch flags removed. English phonemes stay identical to kokoro-onnx's own.

When Kokoro's phoneme output changes, its cache version bumps so old takes miss once; other engines' caches are untouched.

## Readiness checks

Each engine is checked in the background: at launch, when its settings page opens, after Test / Refresh / key / URL / download changes, on engine or preset switch, and after a failed line. VOICEVOX is re-polled while it is failing and in view or in use; Azure is never polled.

- You can still switch to an engine that isn't ready (picker, preset, MCP). While the **speaking** engine is not ready, speech is gated: the Speak button and module controls that speak are off, and Enter on plain text is refused. Slash commands still run so you can switch away. Replay and Regen stay on (frozen engine, may hit cache).
- The voice bar footer shows the failing engine's summary with **Check again** and **Open settings**; the nav marks in-use failing engines with a dot. Nothing navigates to a settings page on its own.
- Provider pages always show connection and local rows (cache, synth ahead, usage); only the voice/character catalog waits behind the check.
- MCP, hotkeys, Live Chat, and web panels are not gated.

## Engine-agnostic UI checklist

- Style and pace chips bind to `Capabilities`, not hardcoded lists.  
- Switching engine or model coerces multi-style selections down to one style when the new target is not multi-style.  
- Marker pickers and alias targets come from the union of adapter catalogs, labeled by kind and source.  
- Provider-specific pages (Test / Refresh / cache / usage) live under Voice Engines; Speak itself stays catalog-driven.
