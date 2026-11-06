# Modules

Modules are **third-party add-ons under Speak**. They speak with the voices MiraSpeak already has (the live chips or a numbered preset). They never add, configure, or pick TTS providers, models, or API keys.

## Form page

A module's page is declarative fields (plain data). The host renders them with its own form builders, so modules supply no widgets, styles, or colors. A `ui.Rows` list with no rows shows the host empty state from `ui.Empty` (title, body, an action that opens a File field's Browse or reaches `on_action`, and art named from `ui.ART_NAMES`), set on the spec with `when_empty` or at runtime with `ctx.ui.set_empty`.

Enabled form pages appear under Speak, above the settings search. Integrations → Modules manages enablement and permissions.

When a module has the `speak` grant, the host pins the same voice bar Speak uses to the bottom of its form page, including the Translate chip and a "now speaking" level meter. Its footer shows `Ctrl+Shift+1-9 preset` and has no control on the right. While the speaking engine isn't ready, module controls marked as speaking are turned off. Modules do not declare or place voice controls: they speak with Live, and code that needs a fixed voice passes Preset *N*.

## Host boundary

- **Python modules** run on their own thread. Calls into the host hop to the GUI thread through a shared bridge with a short timeout.
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
| `observe_playback` | Required for `Module.on_playback_signal(PlaybackSignal)` (frames coalesced to at most 60 fps) and the mouth API (`on_mouth(MouthLine)`); keeps the playback feed running |
| `observe_chat` | Shared Live Chat messages (`on_chat_message`) |
| `stop_all` | Required to stop lines the module does not own |
| `network` / `files` | Consent only: a Python module runs with the same access as MiraSpeak |

Guard rails refuse speech past a waiting-line cap or a call-rate window, and stop a Python module after repeated errors.

## Official modules

These install from GitHub. Paste the URL under **Integrations → Modules → Install from GitHub**. Use the tag `v0.1.0`, or leave the ref blank for the default branch.

| Module | URL | Asks for |
| --- | --- | --- |
| Timer | https://github.com/miramocha/MiraSpeak-timer | speak, MCP tools |
| Lyrics | https://github.com/miramocha/MiraSpeak-lyrics | speak, network, MCP tools |
| EPUB | https://github.com/miramocha/MiraSpeak-epub | speak |

Timer says a phrase on a schedule. EPUB opens a local book and reads it aloud. Lyrics looks up a song on [LRCLIB](https://lrclib.net). Granting network to a Python module is not limited to that host: the module runs with the same access as MiraSpeak.

## Install and consent

A module installs from a .zip, or from a public GitHub repository URL plus a tag, branch, or commit (blank uses the default branch). A GitHub install downloads that commit's archive and runs it through the same .zip checks; there is no git clone. The archive is checked member by member before anything is written: no absolute paths, parent-directory escapes, symbolic links, encrypted entries, or names Windows cannot hold, with caps on total size and file count that are enforced on the bytes actually written. It is extracted to a staging folder, its manifest is validated, and only then is it swapped into the modules folder, with the old copy restored if the swap fails. The manifest sits at the zip root or inside a single top-level folder. A GitHub install records the repository URL and the commit on the Modules row; updating asks for the ref again and does not check in the background. Installing or updating from a .zip clears that record.

Before a new module is enabled, a consent screen lists its permissions in plain words and the exact network hosts it asked for. Python modules are told they run with the same access as MiraSpeak. An update that asks for more than was granted shows the new items marked and stays disabled until approved. Declining installs the files but leaves the module off, and enabling it later asks again.

A bundled module whose `module.toml` sets `default_enabled` starts enabled on first run with its permissions granted, until the user turns it off.

Removing an installed module deletes its files; the user chooses whether to keep or delete its saved data and settings.

## Data layout

Each module has one data root:

- Typed form values, kept separate from the untyped document store
- A JSON document store (atomic writes, size-capped)
- An optional data directory

## Playback feed

While a line plays and a running module holds `observe_playback`, a feed produces the clip's loudness and AEIOU mouth shapes. Analysis runs on the feed thread, never on the playback thread.

Python modules with `observe_playback` get `Module.on_playback_signal(PlaybackSignal)`, with frames coalesced to at most 60 fps. They can also set a mouth profile and receive `on_mouth(MouthLine)` callbacks; each carries a host-built mouth track plus the player-clock position, so the module can follow lip timing on its own thread.
