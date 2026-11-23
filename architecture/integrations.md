# Integrations

Everything here is opt-in and off by default. Each integration reaches the app through the
controller facade, the same surface the UI uses, so it can't bypass engine-agnostic rules or
frozen capture.

## MCP

A local [Model Context Protocol](https://modelcontextprotocol.io/) server lets an AI agent drive
MiraSpeak.

- Streamable HTTP on `127.0.0.1` only, at a configurable port. A port collision fails and doesn't
  block startup. `/mcp` is the full tool set. `/mcp/remote` is the tunnel set.
- MCP is off by default. There is no login on the port: any process on the PC can use every tool
  while MCP is on.
- Tools cover speaking, stopping, presets, live voice, catalogs, themes and appearance,
  translation, rendering speech to a file without playing it, and running a module's own tools.
  The generated tool table, argument shapes, error codes, filler gates, and `render_audio`
  behavior are in [Connect an MCP client](../guides/mcp-clients.md). Render lines are also
  described in [Line lifecycle](line-lifecycle.md#render-lines).
- Switching to an engine that isn't ready succeeds with a warning. MCP lines aren't held back, so
  they can fail until the engine is ready (see [Voice engines](engines.md#readiness-checks)).
- Every tool call hops to the GUI thread through the shared bridge.
- There are no tools for secrets, Argos downloads, or clearing caches.
- To connect a local MCP client, see [Connect an MCP client](../guides/mcp-clients.md). To reach
  it from a cloud agent, see [Remote MCP via Cloudflare](../guides/remote-mcp-cloudflare.md).

## Live chat

A platform-agnostic chat bridge under **Integrations → Live chat**. Modes are exclusive:

| Mode | What it does |
| --- | --- |
| Off | Disconnected. No bridge. |
| Chat → TTS | Speak only messages from the logged-in account (self), as plain text. Slash commands are never run from chat. |
| TTS → Chat | Every successful Speak submit also posts the plain text to the joined channel, as the logged-in account (full paste). Text output captions update separately on play. |

Twitch is the first platform: IRC over TLS with PKCE OAuth through a loopback redirect. New
platforms plug in as adapters without changing the controller.

**Speak as** (under the mode) is **Active** by default, which is the Persona chip, or Manual or a
preset when no persona is on. Pick a persona to speak each Chat → TTS line as that persona. A
`Name:` prefix switches that one line, the same as a typed line; see
[Controls](controls.md#personas). The Persona chip doesn't move. Mute, rate cap, color, and name
follow that persona. If it's deleted, Speak as returns to Active and the page notes that.

1. Open **Live chat**, select **Log in with Twitch** (system browser + PKCE). MiraSpeak ships a
   public Twitch Client ID; redirect URI is `http://localhost:18923/callback`. Channel shows the
   joined login (**Your channel** when empty, which is that account). **Edit** opens the field
   and **Done** checks it: a Twitch login joins that channel, so a bot account can post in a
   streamer's chat, and a bad name isn't saved. Login doesn't overwrite Channel. Tokens stay in
   the OS credential store, never in `settings.toml`.
2. Pick a mode (Settings radios, or the Speak slash command `/livechat off`, `/livechat chat`,
   `/livechat tts`). Chat → TTS ignores other viewers. TTS → Chat posts on submit.

**Share** is gated by `LIVE_CHAT_SHARE` in `app/features.py`. The flag is off: the section is
hidden, and messages aren't published. With the flag on, **Share chat messages** (checkbox off by
default, under the mode and Speak as controls) publishes messages that pass a trigger (mentions,
a command prefix, or all), a who-rule (everyone, subscribers and VIPs, or mods), and a per-user
and global cooldown. It stays connected while on, including when the mode is Off. Chat → TTS
still speaks only your own messages. A message that contains a link waits for the Link hold
(3 seconds by default) so a deletion can arrive first; Skip links drops those messages instead.
Messages without a link aren't delayed. The same event goes to a Python module granted
`observe_chat`, and to an optional webhook on that same Share section (https, or http to
localhost; the header is one `Name: value` line in the credential store).

## Avatar (VMC)

Avatar (VMC) is a bundled [module](modules.md#bundled-modules) (id `avatar`). It's off by
default. Enable it on **Integrations → Modules**, and its page appears under Speak. It asks for
`network`, `observe_lines`, and `observe_playback`. It sends mouth blendshapes over the
[VMC protocol](https://protocol.vmc.info/) while a line plays, to the address in the Connection
section. Only blendshapes are sent, no bones. MiraSpeak analyzes the finished audio (loudness +
spectrum), not typed words, except in the AEIOU text and hybrid sources below.

The Connection section sets **Host** and **Port** (default `127.0.0.1:39539`). Its status line
shows the address it sends to. The page shows live meters for the active keys.

| Mode | Keys |
| --- | --- |
| ARKit (Perfect Sync) | The 8 Perfect Sync mouth keys at 60 fps, PascalCase (`JawOpen`, `MouthFunnel`, and others), analyzed from the audio |
| AEIOU (custom names) | Soft A/I/U/E/O weights sent under shapekey names you enter (for example `Fcl_MTH_A`). Blank vowels are skipped. **AEIOU source** picks how they move: **Spectral** (from the audio), **Text (EN)** (from English spelling), or **Hybrid (EN)** (spelling picks the shape, loudness sets how far the mouth opens). A hint under the dropdown describes the chosen source. |

Three sliders tune the mouth: **Mouth offset** (−200 to 200 ms, default 0), **Mouth gain**
(0.25×–4×, default 1×), and **Mouth smooth** (0–300 ms, default 80; 0 = snap). **Live mouth
shapes** shows the values being sent.

1. Turn on **Avatar (VMC)** under **Integrations → Modules**, then open its page. Set host/port,
   pick a mouth mode, and tune the sliders if needed. For AEIOU, fill the blendshape names your
   receiver expects (case-sensitive).
2. Point a VMC receiver at that host/port for **blendshapes only**, not full-body pose tracking
   (blendshapes-only traffic can look like lost tracking). Keep pose on your usual tracker.
3. Wire the receiver's blendshapes into face tracking (or zero other mouth keys that would fight
   these values). Disable any other lip sync on the same mouth channel.

The module sets a mouth profile and gets a mouth track for each line through the
[mouth API](modules.md#playback-feed). Its own 60 Hz thread follows the player clock. When a
line ends, the mouth closes. Shared clip features are computed on the synth thread before the
clip is queued, while that line still holds its ahead permit. The play thread builds the mouth
track from those features after it releases the permit.

## Modules

Modules add a page under Speak, and some add a voice engine. A module can also expose MCP tools.
They run inside MiraSpeak with the same access to your PC, so only install modules from people
you trust. Official module URLs, permissions, and installer behavior are in
[Modules](modules.md#official-modules).

1. Open **Integrations → Modules** and choose **Install from .zip…**. You see the permissions it
   asks for before it's enabled. **Install, keep off** installs it disabled. Installed modules
   live in `%LOCALAPPDATA%\MiraSpeak\modules\<id>`. Module authors can use **Load from folder…**
   to run one from its source folder (**Reload** picks up edits).

   **Install from URL…** takes a public GitHub repository. Paste
   `https://github.com/<owner>/<repo>` or a folder link
   `https://github.com/<owner>/<repo>/tree/<ref>/<folder>`. The link rules and how to publish a
   module are in [Install a module from GitHub](../guides/module-github-install.md).
2. **Update…** on a module's row takes a newer .zip of the same module (for a GitHub module, a
   new tag or commit of the same repo and folder). If it asks for more permissions, it stays off
   until you approve them. **Remove** can keep or delete the module's saved data.
3. Turn a module on and grant the permissions it asks for (speak, see other lines, stop all,
   network, files, MCP tools). Changing a grant restarts it.

Reference modules are one Git submodule at `examples/modules/`. Clone with
`git clone --recurse-submodules`, or run `git submodule update --init` after a plain clone.
