# Integrations

Everything here is opt-in and off by default. Each integration reaches the app through the controller facade, the same surface the UI uses, so it can't bypass engine-agnostic rules or frozen capture.

## MCP

A local [Model Context Protocol](https://modelcontextprotocol.io/) server lets an AI agent drive MiraSpeak.

- Streamable HTTP on `127.0.0.1` only, at a configurable port. A port collision fails softly and doesn't block startup.
- Tools cover speaking, stopping, presets and preset sets, the live voice chips (engine, model, voice, styles, pace, language, pitch), catalogs, themes and appearance (including Font Set and font weight), translation (`set_translation`, `list_translation_languages`, and translation state in `get_status`), rendering speech to an audio file without playing it (`render`; see [Line lifecycle](line-lifecycle.md#render-lines)), and running modules' own tools.
- `speak` accepts the same leading slash commands as the line box, except `/file` and `/render`, which open a dialog on the PC. Use `speak_from_file` and `render` instead.
- Switching to an engine that isn't ready succeeds with a warning; speech then waits until the engine is ready (see [Voice engines](engines.md#readiness-checks)).
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

- **ARKit mode:** Perfect Sync mouth keys at 60 fps, analyzed from the audio.
- **AEIOU mode:** user-named vowel keys, driven by the audio, by English text, or by a hybrid of the two.
- Gain, smoothing, and a timing offset are adjustable, with live meters for preview.
- Mouth analysis runs on the playback path after the next synthesis is unblocked, and the mouth zeroes when a line stops.

## Text Output

Writes the currently playing line to a text file for stream overlays. The write is atomic, and the file clears a couple of seconds after playback goes idle. When a line was [translated](translation.md), the caption can be the translated text (default), the original, or both.

## Quick bar and hotkeys

A floating composer summoned with a global hotkey, plus `Ctrl+Shift+1..9` to apply presets. Hotkeys are registered with the OS (`RegisterHotKey`) without low-level keyboard hooks. The window and overlay can each be hidden from screen capture.
