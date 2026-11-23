# Modules

Modules add a page, usually under Speak, and some add a voice engine. A speak-page module uses the voices MiraSpeak already has (the live chips or a numbered preset) and doesn't pick or reconfigure those voices. An [engine module](#engine-modules) can add a voice engine of its own, such as Piper. How to install from a .zip, a folder, or a GitHub URL is in [Integrations](integrations.md#modules). To write a module, see the [Module API](../module-api/README.md).

## Form page

A module's page is declarative fields (plain data). The host renders them with its own form builders, so modules supply no widgets, styles, or colors. A `ui.Rows` list with no rows shows the host empty state from `ui.Empty` (title, body, an action that opens a File field's Browse or reaches `on_action`, and art named from `ui.ART_NAMES`), set on the spec with `when_empty` or at runtime with `ctx.ui.set_empty`. `ui.Slider` is a number field drawn as a slider with a live value label. `ui.Meters` is a read-only block of 0–1 bars that the module fills with `ctx.ui.set_meters`.

Enabled form pages appear under Speak, above the settings search. An engine module's page sits on Voice engines instead. Integrations → Modules manages enablement and permissions.

When a module has the `speak` grant, the host pins the same voice bar Speak uses to the bottom of its form page, including the Translate chip and a "now speaking" level meter. Its footer shows `Ctrl+Shift+1-9 preset` and has no control on the right. While the speaking engine isn't ready, module controls marked as speaking are turned off. Modules don't declare or place voice controls: they speak with Live, and code that needs a fixed voice passes Preset *N*.

## Host boundary

- Python modules run on their own thread. Calls into the host hop to the GUI thread through a shared bridge with a short timeout.
- Each runtime gets its own bound host. A stopped or replaced runtime is refused for calls and ignored for posts.

## Speech

Modules queue speech through the same path as the Speak page, with:

- A voice reference: Live (current chips) or Preset *N* (the preset's frozen voice; an empty slot is refused).
- A line source: the module id and display name, used for ownership, the Speak row subtitle, and "stop my lines".

Modules never inject slash commands and never change the live chips as a side effect of speaking.

## Permissions

| Permission | Enforcement |
| --- | --- |
| `speak` | Required to queue speech |
| `observe_lines` | Required to see other sources' line and playback events |
| `observe_playback` | Required for `Module.on_playback_signal(PlaybackSignal)` (frames coalesced to at most 60 fps) and the mouth API (`on_mouth(MouthLine)`); keeps the playback feed running |
| `observe_chat` | Shared Live chat messages (`on_chat_message`) |
| `stop_all` | Required to stop lines the module doesn't own |
| `audio` | Required to receive the audio of every line MiraSpeak speaks |
| `style_lines` | Required to add one voice style tag to a live line before it is spoken |
| `mcp_tools` | Required to add tools and instructions to the MiraSpeak MCP server |
| `filler` | Required to play cached filler clips for a persona |
| `voice_engine` | Required to add voice engines (`Module.engines()`) and to read or change their settings |
| `network` / `files` | Consent only: a Python module runs with the same access as MiraSpeak |

Guard rails refuse speech past a waiting-line cap or a call-rate window, and stop a Python module after repeated errors.

## Official modules

These install from GitHub. On **Integrations → Modules**, select **Install from URL…**. In the Install from GitHub dialog, paste the link. The link sets the tag and the folder.

| Module | URL | Asks for |
| --- | --- | --- |
| Timer | https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/timer | speak, mcp_tools |
| Lyrics | https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/lyrics | speak, network, mcp_tools |
| EPUB | https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/epub | speak |
| Agent Chat | https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/agentchat | network, filler |
| Discord | https://github.com/miramocha/MiraSpeak-modules/tree/v0.2.0/discord | audio, network, observe_lines; installs `discord.py[voice]` and `davey` |
| Piper | https://github.com/miramocha/MiraSpeak-modules/tree/v0.3.0/piper | voice_engine, network; installs `piper-tts` and `pathvalidate` |

Timer says a phrase on a schedule. EPUB opens a local book and reads it aloud. Lyrics looks up a song on [LRCLIB](https://lrclib.net). Agent Chat sends what you type to an agent and plays a filler while you wait. The agent speaks the reply. Discord plays everything MiraSpeak speaks in your voice channel through your own bot; setup is in the [Discord guide](../guides/discord.md). Piper adds local English Piper voices; see the [Piper guide](../guides/piper.md). Granting network to a Python module isn't limited to that host: the module runs with the same access as MiraSpeak.

## Engine modules

A module with the `voice_engine` permission adds voice engines. `Module.engines()` returns `Adapter` classes. MiraSpeak calls it once on the module thread after `on_start`.

| Rule | What happens |
| --- | --- |
| Adding and dropping | MiraSpeak adds the engines once the module is running and drops them when it stops, is disabled, or is removed. An engine id that MiraSpeak or another module already uses is skipped with a warning. |
| Settings | The adapter's `default_config` fills in missing keys of its `[engines.<id>]` table when the engine is added. The table stays when the module stops, so a restart picks up where it left off. |
| Voice list | `ctx.engine_config(id)` returns a copy of that table. `ctx.set_engine_config(id, voices=…)` replaces the voice list (Speak alias → the engine's own voice id). When the live voice is no longer in it, MiraSpeak switches to the first alias. Both work only for the module's own engines. |
| Threads | Adapters aren't module callbacks. MiraSpeak creates them and calls `synthesize` on its synthesis threads, not the module thread, so adapter state must be thread-safe and must not call `ModuleContext`. |
| API | Engine modules build adapters from `miraspeak.modules.engine_api`: `Adapter`, `Capabilities`, `SynthesisResult`, the style and pace helpers, the audio and download helpers, and `engine_data_dir(id)` for downloads (`<data>/<engine id>`, the folder the built-in engines use). `ENGINE_API_VERSION` goes up when a name there changes in a way an engine module would notice. The module `API_VERSION` stays 1. |
| Page | An engine module's form page sits in the Voice engines Engine list (In-app), placed by name, not under Speak. MiraSpeak adds its own Speech cache and Generate ahead rows under the form. The engine gets the same readiness check as a built-in one, from its adapter's `unavailable_reason`, so the list and nav show the warning dot and reason. |
| Missing engine | A line that names an engine whose module isn't running speaks with the [fallback engine](engines.md#engine-modules). |

## Bundled modules

These ship with MiraSpeak. Enable them on Integrations → Modules. Each page appears under Speak.

| Module | Description | Asks for |
| --- | --- | --- |
| Avatar (VMC) | "Sends mouth shapes to a VMC receiver while MiraSpeak speaks, so your avatar's lips follow the voice." See [Integrations](integrations.md#avatar-vmc). | network, observe_lines, observe_playback |
| Emotion | "Adds one Azure style tag to a live English line from a local emotion classifier." | style_lines, network |

## Install and consent

A module installs from a .zip, or from a public GitHub repository URL plus a tag, branch, or commit (blank uses the default branch). A GitHub install downloads that commit's archive and runs it through the same .zip checks; there is no git clone. The archive is checked member by member before anything is written: no absolute paths, parent-directory escapes, symbolic links, encrypted entries, or names Windows can't hold, with caps on total size and file count that are enforced on the bytes actually written. It's extracted to a staging folder, its manifest is validated, and only then is it swapped into the modules folder, with the old copy restored if the swap fails. The manifest sits at the zip root or inside a single top-level folder. A GitHub install records the repository URL and the commit on the Modules row; updating asks for the ref again and doesn't check in the background. Installing or updating from a .zip clears that record.

Before a new module is enabled, a consent screen lists its permissions in plain words and the exact network hosts it asked for. Python modules are told they run with the same access as MiraSpeak. An update that asks for more than was granted shows the new items marked and stays disabled until approved. Declining installs the files but leaves the module off, and enabling it later asks again.

### Module packages

A module can list Python packages in `requires` as exact pins, such as `requires = ["discord.py[voice]==2.7.1"]`. It ships one `pylock.<python>-<platform>.toml` per target with the wheel URLs and sha256 hashes. The file rules are in [Module manifest](../module-api/core/manifest.md#packages), and the steps are in [Build pylock files](../guides/module-packages.md).

**Consent:** the consent screen lists the packages. Declining installs nothing, and an update that changes the packages asks again.

**Install:** after approval, MiraSpeak downloads the wheels for this machine from PyPI on a worker thread. It checks each hash and unpacks the wheels into the module's own `site` folder before the module is swapped in. A failed download, a hash mismatch, or a missing lock for this platform installs nothing. MiraSpeak needs no pip or Python of its own for this. CetiModule does the install.

**Loading:** `site` goes at the end of the import path when the module loads, so a package MiraSpeak already has wins.

**Windows:** Windows holds a loaded compiled package open. An update or removal that it blocks finishes at the next start, and the result says to restart.

A bundled module whose `module.toml` sets `default_enabled` starts enabled on first run with its permissions granted, until the user turns it off.

Removing an installed module deletes its files; the user chooses whether to keep or delete its saved data and settings.

## Data layout

Each module keeps two kinds of data:

| Kind | Where it lives |
| --- | --- |
| Form values | Typed page values in `module_settings/<id>.json` |
| Data files | The module's own text files in `module_data/<id>/`, written with `ctx.data` ([Author API](../module-api/core/author-api.md#data-files)) |

## Playback feed

While a line plays and a running module holds `observe_playback`, a feed produces the clip's loudness and AEIOU mouth shapes. Analysis runs on the feed thread, never on the playback thread.

Python modules with `observe_playback` get `Module.on_playback_signal(PlaybackSignal)`, with frames coalesced to at most 60 fps. They can also set a mouth profile with `ctx.set_mouth_profile(MouthProfile)` and receive `on_mouth(MouthLine)` callbacks; each carries a host-built mouth track plus the player-clock position, so the module can follow lip timing on its own thread.

A `MouthProfile` has a mode (`arkit` or `aeiou`). For `aeiou`, it also has a source (`audio`, `text`, or `hybrid`) and the module's key names. `arkit` uses the 8 Perfect Sync keys and ignores those names. `ctx.set_mouth_profile` doesn't wait for MiraSpeak, so it's safe in `on_start`. A missing permission is logged, not raised. Passing `None` stops `on_mouth`. It doesn't stop `on_playback_signal`.
