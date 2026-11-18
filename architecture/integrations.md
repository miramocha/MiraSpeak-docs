# Integrations

Everything here is opt-in and off by default. Each integration reaches the app through the controller facade, the same surface the UI uses, so it can't bypass engine-agnostic rules or frozen capture.

## MCP

A local [Model Context Protocol](https://modelcontextprotocol.io/) server lets an AI agent drive MiraSpeak.

- Streamable HTTP on `127.0.0.1` only, at a configurable port. A port collision fails and doesn't block startup. The endpoint is `/mcp`.
- MCP is off by default. There is no login on the port: any process on the PC can use every tool while MCP is on.
- Tools cover speaking, stopping, presets and preset sets, the live voice chips, catalogs, themes and appearance (including Font Set and font weight), translation (`set_translation`, `list_translation_languages`, and translation state in `get_status`), rendering speech to an audio file without playing it (`render`; see [Line lifecycle](line-lifecycle.md#render-lines)), and running modules' own tools. Tool names, arguments, and replies are in [Connect an MCP client](../guides/mcp-clients.md).
- `speak` takes `text` and returns `{ok, queue_depth}` when the line is queued. Slash commands in that text still run, except commands that add modules or open a file dialog, which are refused; call `speak_from_file` or `render` instead. Only the person at the PC can quit or restart the app, or add modules.
- `stop` takes `scope` (`current` or `all`).
- Switching to an engine that isn't ready succeeds with a warning. MCP lines aren't held back, so they can fail until the engine is ready (see [Voice engines](engines.md#readiness-checks)).
- Live voice setters enter Manual. `update_preset` and the theme / appearance tools don't. `set_translation` doesn't enter Manual.
- Catalog tools fail with `reason=unknown_engine` for an engine that doesn't exist. Bad hex keys in `set_appearance` are skipped.
- A failure is `{ok: false, reason, message}` plus any extra fields. Clients match on `reason`.
- Every tool call hops to the GUI thread through the shared bridge.
- There are no tools for secrets, Argos downloads, or clearing caches.
- To connect a local MCP client, see [Connect an MCP client](../guides/mcp-clients.md). To reach it from a cloud agent, see [Remote MCP via Cloudflare](../guides/remote-mcp-cloudflare.md) (not in the current build yet).

## Live chat

A platform-agnostic chat bridge with one exclusive mode at a time:

| Mode | Behavior |
| --- | --- |
| **Chat → TTS** | Your own chat messages are spoken (as plain text; slash commands are never run from chat). |
| **TTS → Chat** | Lines you speak are posted to chat when they start playing. |
| **Off** | No bridge. |

Twitch is the first platform: IRC over TLS with PKCE OAuth through a loopback redirect. Tokens live in the OS credential store. New platforms plug in as adapters without changing the controller. The mode can also be switched with the `/livechat` command.

**Share** is gated by `LIVE_CHAT_SHARE` in `app/features.py`. The flag is off: the section is hidden, and messages aren't published. With the flag on, **Share chat messages** (checkbox off by default, under the mode and Speak as controls) publishes messages that pass a trigger (mentions, a command prefix, or all), a who-rule (everyone, subscribers and VIPs, or mods), and a per-user and global cooldown. It stays connected while on, including when the mode is Off. Chat → TTS still speaks only your own messages. A message that contains a link waits for the Link hold (3 seconds by default) so a deletion can arrive first; Skip links drops those messages instead. Messages without a link aren't delayed. The same event goes to a Python module granted `observe_chat`, and to an optional webhook on that same Share section (https, or http to localhost; the header is one `Name: value` line in the credential store).

## Avatar (VMC)

Avatar (VMC) is a bundled [module](modules.md#bundled-modules) (id `avatar`). Enable it on Integrations → Modules, and its page appears under Speak. It asks for `network`, `observe_lines`, and `observe_playback`. It sends mouth blendshapes over the [VMC protocol](https://protocol.vmc.info/) to the address in the Connection section. Only blendshapes are sent, no bones.

- The Connection section sets **Host** and **Port** (default `127.0.0.1:39539`). Its status line shows the address it sends to.
- In **ARKit (Perfect Sync)** mode, it sends the 8 Perfect Sync mouth keys at 60 fps, analyzed from the audio.
- In **AEIOU (custom names)** mode, it sends the vowel keys named under **AEIOU shapekeys**. **AEIOU source** picks how they move: **Spectral** (from the audio), **Text (EN)** (from English spelling), or **Hybrid (EN)** (spelling picks the shape, loudness sets how far the mouth opens). A hint under the dropdown describes the chosen source.
- Three sliders tune the mouth: **Mouth offset** (−200 to 200 ms, default 0), **Mouth gain** (0.25x to 4x, default 1x), and **Mouth smooth** (0 to 300 ms, default 80). **Live mouth shapes** shows the values being sent.
- The module sets a mouth profile and gets a mouth track for each line through the [mouth API](modules.md#playback-feed). Its own 60 Hz thread follows the player clock. When a line ends, the mouth closes.
- Shared clip features are computed on the synth thread before the clip is queued, while that line still holds its ahead permit. The play thread builds the mouth track from those features after it releases the permit.
