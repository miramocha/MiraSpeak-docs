# Layers

Each layer has a clear job. Higher layers may call lower ones; the reverse is forbidden—even under type-checking imports.

## `core/`

Line text and speak commands, with no I/O to engines or UI.

- Tags and markers scrub or keep bracket tokens and expand marker aliases (`[-name]` → catalog id). Styles aren't parsed from mid-line brackets into chip state. MiraSpeak-wide `[date]` / `[time]` stay in stored line text and expand at synth/play from the local clock (`core/dynamic_markers.py`). `[time]` is hour and minute; `[time:full]` adds seconds; `[time:24h]` is `HH:MM`; `[time:tod]` is the day period (empty on 24-hour locales); `[time:h]` / `[time:m]` / `[time:s]` are digits.
- Commands are the start-of-line slash catalog (`/presetN`, `/stop`, `/file`, `/render`, …) and user bindings (alias, fixed phrase, or live file). Mid-line `/…` is ordinary speech text. Leading `//` escapes.
- Argument commands such as `/theme`, `/preset`, `/presetset`, `/engine`, `/split`, `/livechat`, `/translate`, and the chip commands `/character`, `/voice`, `/speech`, `/style`, `/pace`, and `/pitch` take a choice instead of speech. Once the token is confirmed, the message field becomes a search over that command's choices; the remainder is matched (exact value, exact label, then label prefix, word prefix, substring) and never spoken. Chip commands enter Manual, and `/style` replaces the selection. A chip command is left out of the `/` list while the engine hides that chip, and typing it shows a note such as "Kokoro has no styles." An alias on Voice → Commands can fix the value (Mode: Value, for example, `/loud` → `/pitch +2`), which runs at once without a search. Adding one is data: a spec plus a choice source in the controller.

## `engines/`

TTS adapters behind a shared contract (`Capabilities`, synthesize → audio clip).

- Catalogs for models, voices, styles, paces, speech languages, and inline markers.
- Registry of shipped adapters; availability checks stay off the UI thread where they involve network or heavy work.
- Pure clip helpers that adapters need, such as concat, live here, not in `audio/`, so engines never depend upward.

## `audio/` and `mouth/`

- `audio/` holds device enumeration (Output starts on the system default device), formant-preserving pitch, and dual-output playback. Audio devices can create a virtual output. On Windows that's VB-Cable. On Linux it's a PulseAudio null sink. OBS takes the voice from that device.
- `mouth/` builds mouth tracks from a finished clip or its text, for modules and the player. It has no Qt. VMC output is the bundled [Avatar (VMC)](integrations.md#avatar-vmc) module, not a package.

## `app/`

Qt-free application services:

- Speak service and synth pipeline (queues, ahead permits, cache short-circuit, play handoff).
- Settings load/save (atomic writes; type coercion; unknown keys ignored).
- Line status model (queued → synthesizing → ready → playing → terminal).
- Secrets via keyring; speech cache policy; latest-line / session file helpers.
- Live chat bridge (`app/chat`)—platform-agnostic; Twitch is the first adapter.
- Translation (`app/translation`)—providers, language detection, marker-safe segmenting, and the SQLite translation cache. See [Translation](translation.md).
- Beta access (`app/beta_access`)—GitHub device-flow sign-in and the beta-list check for beta builds. See [Startup](overview.md#startup).

## `mcp/`

Optional local MCP server (loopback only). Tools reach the running app through the controller facade—no keyring tools, no direct settings mutation beyond what the facade allows.

## `modules/`

Qt-free module runtime: manifests, loader, permissions, playback feed. See [Modules](modules.md).

## `ui/`

Qt widgets and controller facades. The design system under this layer is the `cetiui` package: theme tokens, bundled themes and fonts, the base stylesheet, and the generic widgets. MiraSpeak's `ui/` keeps the app sheet, the app widgets, and the controller facades.

- Speak page (recent lines, composer), settings pages, and the quick bar.
- `AppController` and facades (`azure`, `themes`, `modules`, …) are the only API widgets use.
- Widgets may import plain value types and constants from lower layers; they must not drive engines or I/O modules themselves.
