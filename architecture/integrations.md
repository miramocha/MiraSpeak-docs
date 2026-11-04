# Integrations

Everything here is opt-in and off by default. Each integration reaches the app through the controller facade, the same surface the UI uses, so it can't bypass engine-agnostic rules or frozen capture.

## MCP

A local [Model Context Protocol](https://modelcontextprotocol.io/) server lets an AI agent drive MiraSpeak.

- Streamable HTTP on `127.0.0.1` only, at a configurable port. A port collision fails softly and doesn't block startup.
- Tools cover speaking, stopping, presets and preset sets, the live voice chips (engine, model, voice, styles, pace, language, pitch), catalogs, themes and appearance (including Font Set and font weight), translation (`set_translation`, `list_translation_languages`, and translation state in `get_status`), rendering speech to an audio file without playing it (`render`; see [Line lifecycle](line-lifecycle.md#render-lines)), and running modules' own tools.
- `speak` accepts the same leading slash commands as the line box, except `/file` and `/render`, which open a dialog on the PC. Use `speak_from_file` and `render` instead. `/loadmodule` and `/installmodule` are refused too: only the person at the PC can add modules.
- `speak` and `speak_from_file` return `line_ids`, one per queued line (several with Split Sentences), in order. Both take an optional `source` tag (1–64 characters); the lines and their events carry it as `mcp:<tag>`.
- `stop` takes `scope` (`current` or `all`), or `line_ids` and/or `source` to stop only those lines while the rest play on. With either one set, `scope` is ignored.
- Switching to an engine that isn't ready succeeds with a warning; speech then waits until the engine is ready (see [Voice engines](engines.md#readiness-checks)).
- `GET /events` on the same port is a read-only Server-Sent Events stream of line events: `queued`, `playing`, `done`, `failed`, `stopped` (internal `synthesizing` / `ready` are not sent). Each message is the shared envelope `{app, type, id, ts, data}`; `data` has `line_id` (the ids `speak` returns), `text`, and when they apply `source`, `batch_*`, `silent`, `duration_s` (playing), `error` (failed). `id` rises for the whole app run; reconnect with `Last-Event-ID` (or `?after=`) to replay up to the last 500 events. A client that falls 256 events behind is disconnected and can resume the same way. Non-loopback `Host` headers get 403.
- Every tool call hops to the GUI thread through the shared bridge. Failures come back as structured `{ok, reason, message}` results.
- Changing a live voice setting switches to Manual (no active preset); editing a preset does not.
- There are no tools for secrets, Argos downloads, or clearing caches.
- To reach it from a cloud agent, see [Remote MCP via Cloudflare](../guides/remote-mcp-cloudflare.md).

## Live Chat

A platform-agnostic chat bridge with one exclusive mode at a time:

| Mode | Behavior |
| --- | --- |
| **Chat → TTS** | Your own chat messages are spoken (as plain text; slash commands are never run from chat). |
| **TTS → Chat** | Lines you speak are posted to chat when they start playing. |
| **Off** | No bridge. |

Twitch is the first platform: IRC over TLS with PKCE OAuth through a loopback redirect. Tokens live in the OS credential store. New platforms plug in as adapters without changing the controller. The mode can also be switched with the `/livechat` command.

## Avatar (VMC)

Optional mouth blendshapes sent over the [VMC protocol](https://protocol.vmc.info/) to a local avatar app. Only blendshapes are sent, no bones.

This is a bundled [module](modules.md) (id `avatar`), not an Output page. Turn it on under **Integrations → Modules**; its page then appears under Speak. It asks for `network`, `observe_lines`, and `observe_playback`. It is off by default. Anyone who had VMC turned on in an older version gets the module turned on once, with their old host, port, mode, and tuning copied over.

- **Connection:** host and port (default `127.0.0.1:39539`), with a status line.

- **ARKit mode:** Perfect Sync mouth keys at 60 fps, analyzed from the audio.
- **AEIOU mode:** user-named vowel keys, driven by the audio (Spectral), by English text, or by a hybrid of the two. A hint under the AEIOU source dropdown says what the chosen source does.
- Mouth offset (±200 ms), gain (0.25×–4×, default 1×), and smoothing (0–300 ms, default 80) are sliders, with live meters for the shapes being sent.
- The module sets a mouth profile and gets a mouth track for each line through the [mouth API](modules.md#playback-feed), then follows the player clock on its own 60 Hz thread. Mouth analysis runs on the playback path after the next synthesis is unblocked, and the mouth zeroes when a line stops.

## Text Output

Writes the currently playing line to a text file for stream overlays. The write is atomic, and the file clears a couple of seconds after playback goes idle. When a line was [translated](translation.md), the caption can be the translated text (default), the original, or both.
