# Modules

Modules are **third-party add-ons under Speak**. They speak with the voices MiraSpeak already has (the live chips or a numbered preset). They never add, configure, or pick TTS providers, models, or API keys.

## Surfaces

| Surface | Role |
| --- | --- |
| **Form page** | Declarative fields (plain data). The host renders them with its own form builders, so modules supply no widgets, styles, or colors. |
| **Web panel** | Full-bleed Chromium page served from a private `miramod://` scheme, with a fenced network policy and a small JS SDK (`window.miraspeak`). |
| **Form + web** | A form above a web panel on the same page. |
| **Browser Source** | Optional localhost stream of the same page for OBS and similar tools, behind a tokenized URL. |

Enabled module pages appear under Speak, above the settings search. Integrations → Modules manages enablement, permissions, and Browser Source.

## Host boundary

- **Python modules** run on their own thread. Calls into the host hop to the GUI thread through a shared bridge with a short timeout.
- **Web panels** already run on the GUI thread and talk to the host over a web channel.
- Both paths reach the same modules controller for speak, stop, voices, and stored values, so permission and rate checks cannot diverge.
- Each runtime gets its own bound host. A stopped or replaced runtime is refused for calls and ignored for posts.

## Speech

Modules queue speech through the same path as the Speak page, with:

- **A voice reference:** Live (current chips) or Preset *N* (the preset's frozen voice; an empty slot is refused).
- **A line source:** the module id and display name, used for ownership, the Speak row subtitle, and "stop my lines".

Modules never inject slash commands and never change the live chips as a side effect of speaking.

## Permissions

| Permission | Enforcement |
| --- | --- |
| `speak` | Required to queue speech |
| `observe_lines` | Required to see other sources' line and playback events |
| `stop_all` | Required to stop lines the module does not own |
| `network` / `files` | Consent only for Python; **enforced** for web panels (connect allowlist and picked-file tokens) |

Guard rails refuse speech past a waiting-line cap or a call-rate window, stop a Python module after repeated errors, and disable a web module after repeated renderer crashes.

## Web fence

One Qt-free policy module owns path resolution, the content security policy, connect-host matching, and file tokens. The scheme handler, the request interceptor, and the Browser Source server all call it. Only the module's own scheme, `data:`, `blob:`, and `about:` are allowed by default; `http(s)` and `ws(s)` need the `network` grant plus a declared host. Grants are fixed for a panel's life, so changing them restarts the module.

## Data layout

Each module has one data root:

- Typed form values, kept separate from the untyped document store
- A web profile for its Chromium panel
- A JSON document store (atomic writes, size-capped)
- Picked-file tokens
- An optional data directory

## Playback feed

While a line plays, an optional feed pushes up to 60 ticks per second with the clip's loudness and AEIOU mouth shapes to interested panels and Browser Source pages. Analysis runs on the feed thread, never on the playback thread. Each viewer can apply its own playback offset, and the feed only runs while some web panel exists.
