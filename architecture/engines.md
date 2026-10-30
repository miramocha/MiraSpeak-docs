# Voice engines

MiraSpeak speaks through **adapters**. The UI never embeds provider-specific style or voice tables; it reads `Capabilities` from the active adapter (and model, where the provider has one).

## Adapter contract (conceptual)

Each adapter exposes:

- Whether it is local or cloud, and whether it is currently configured  
- Models (if any), voices, styles, paces, speech languages  
- Inline marker catalog (what `[id]` tokens survive scrubbing); the registry always adds MiraSpeak `[date]` / `[time]` on top (`[time:full]` / `[time:24h]` / `[time:tod]` / `[time:h|m|s]` share the time keep id)  
- `multi_style` — checklist Style picker vs single-select  
- `synthesize(request) →` audio clip  

Unsupported styles selected in the UI degrade to that engine’s default.

## Shipped engines

| Engine | Where | Notes |
| --- | --- | --- |
| **Kokoro** | Local (ONNX) | Offline synthesis; styles/paces from adapter capabilities. |
| **VOICEVOX** | Local HTTP | Expects a running VOICEVOX engine. Character + style (mood) selection; catalog labels are user-facing. |
| **Azure Speech** | Cloud | One Azure settings surface with a **model** chip (e.g. Dragon HD / Omni / Neural). Multi-style checklist where the model supports it. Secrets in keyring. Optional usage meter and limit action. Setup: [Azure guide](../guides/azure-speech-translator.md). |

Engines share the same submit → synth → play path. Synth-ahead and cache policies are configured per engine id (Azure models share one Azure bucket).

## Engine-agnostic UI checklist

- Style and pace chips bind to `Capabilities`, not hardcoded lists.  
- Switching engine or model coerces multi-style selections down to one style when the new target is not multi-style.  
- Marker pickers and alias targets come from the union of adapter catalogs, labeled by kind and source.  
- Provider-specific pages (Test / Refresh / cache / usage) live under Voice Engines; Speak itself stays catalog-driven.
