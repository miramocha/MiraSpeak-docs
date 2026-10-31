# Modules

Modules are **third-party add-ons under Speak**. They speak with the voices MiraSpeak already has (the live chips or a numbered preset). They never add, configure, or pick TTS providers, models, or API keys.

## Surfaces

| Surface | Role |
| --- | --- |
| **Form page** | Declarative fields (plain data). The host renders them with its own form builders, so modules supply no widgets, styles, or colors. A `ui.Rows` list with no rows shows the host empty state from `ui.Empty` (title, body, an action that opens a File field's Browse or reaches `on_action`, and art named from `ui.ART_NAMES`), set on the spec with `when_empty` or at runtime with `ctx.ui.set_empty`. |
| **Web panel** | Full-bleed Chromium page served from a private `miramod://` scheme, with a fenced network policy and a small JS SDK (`window.miraspeak`). |
| **Form + web** | A form above a web panel on the same page. |
| **Browser Source** | Optional localhost stream of the same page for OBS and similar tools, behind a tokenized URL. |

Enabled module pages appear under Speak, above the settings search. Integrations → Modules manages enablement, permissions, and Browser Source.

When a module has the `speak` grant, the host pins the same voice bar Speak uses to the bottom of its page (form, web, or both), minus the Translate chip. The bar ends with a "now speaking" level meter. Its footer shows `Ctrl+Shift+1-9 preset` and has no control on the right. While the speaking engine isn't ready, module controls marked as speaking are turned off. Modules do not declare or place voice controls: they speak with Live, and code that needs a fixed voice passes Preset *N*.

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

## Install and consent

A module installs from a .zip. The archive is checked member by member before anything is written: no absolute paths, parent-directory escapes, symbolic links, encrypted entries, or names Windows cannot hold, with caps on total size and file count that are enforced on the bytes actually written. It is extracted to a staging folder, its manifest is validated, and only then is it swapped into the modules folder, with the old copy restored if the swap fails. The manifest sits at the zip root or inside a single top-level folder.

Before a new module is enabled, a consent screen lists its permissions in plain words and the exact network hosts it asked for. Python modules are told they run with the same access as MiraSpeak; web-only modules are told what the fence allows; modules with a Browser Source page are told anyone with the stream URL can see its events and saved values. An update that asks for more than was granted shows the new items marked and stays disabled until approved. Declining installs the files but leaves the module off, and enabling it later asks again. Installed web modules reach only the hosts that were granted.

Removing an installed module deletes its files; the user chooses whether to keep or delete its saved data and settings.

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
