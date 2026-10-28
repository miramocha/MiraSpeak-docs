# Layers

Each layer has a clear job. Higher layers may call lower ones; the reverse is forbidden—even under type-checking imports.

## `core/`

Line text and speak commands, with no I/O to engines or UI.

- **Tags / markers** — Scrub or keep bracket tokens; expand marker aliases (`[-name]` → catalog id). Styles are not parsed from mid-line brackets into chip state. MiraSpeak-wide `[date]` / `[time]` stay in stored line text and expand at synth/play from the local clock (`core/dynamic_markers.py`).
- **Commands** — Start-of-line slash catalog (`/presetN`, `/stop`, `/file`, …) and user bindings (alias, fixed phrase, or live file). Mid-line `/…` is ordinary speech text. Leading `//` escapes.

## `engines/`

TTS adapters behind a shared contract (`Capabilities`, synthesize → audio clip).

- Catalogs for models, voices, styles, paces, speech languages, and inline markers.
- Registry of shipped adapters; availability checks stay off the UI thread where they involve network or heavy work.
- Pure clip helpers that adapters need (e.g. concat) live here—not in `audio/`—so engines never depend upward.

## `audio/` and `avatar/`

- **Audio** — Device enumeration, formant-preserving pitch, dual-output playback.
- **Avatar** — Optional VMC mouth blendshapes (ARKit or AEIOU profiles). Mouth analysis runs on the play path so it does not block the next synthesis.

## `app/`

Qt-free application services:

- Speak service and synth pipeline (queues, ahead permits, cache short-circuit, play handoff).
- Settings load/save (atomic writes; type coercion; unknown keys ignored).
- Line status model (queued → synthesizing → ready → playing → terminal).
- Secrets via keyring; speech cache policy; latest-line / session file helpers.
- Live Chat bridge (`app/chat`) — platform-agnostic; Twitch is the first adapter.

## `mcp/`

Optional local MCP server (loopback only). Tools reach the running app through the controller facade—no keyring tools, no direct settings mutation beyond what the facade allows.

## `modules/`

Qt-free module runtime: manifests, loader, permissions, web policy, Browser Source server, playback feed. See [Modules](modules.md).

## `ui/`

Qt widgets and controller facades.

- Speak page (recent lines, composer), settings pages, quick bar / overlay.
- `AppController` and facades (`azure`, `themes`, `modules`, …) are the only API widgets use.
- Widgets may import plain value types and constants from lower layers; they must not drive engines or I/O modules themselves.
