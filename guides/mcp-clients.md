# Connect an MCP client

MiraSpeak's built-in [MCP](https://modelcontextprotocol.io/) server lets an MCP client on the
same PC drive the running app: speak lines, change voices and presets, render audio, and call
module tools. This page covers enabling the server, connecting a client, and the tool reference.
For a cloud agent that can't reach your PC, see [Remote MCP via Cloudflare](remote-mcp-cloudflare.md).

## Enable the server

1. Open **MCP** under Integrations. Check **Enable MCP server (localhost only)**, set **Port**
   (default `56769`). Toggle off anytime to stop listening. No restart is needed. If the port is
   taken, MiraSpeak stays up and shows an error; pick another port. There is no login: while MCP
   is on, any process on this PC can use every tool, including deleting themes and presets, and
   quitting the app. `speak_from_file` reads only the folders you list under **File folders** on
   that page; empty refuses every path.

2. Point Cursor at the same port in `mcp.json` (project or `~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "miraspeak": {
      "url": "http://127.0.0.1:56769/mcp"
    }
  }
}
```

The footer should read `Listening on http://127.0.0.1:<port>/mcp`.

The server is Streamable HTTP, bound to `127.0.0.1` only. There is no stdio transport. Use
`127.0.0.1` in client config, not `localhost` (which can resolve to IPv6).

If you changed the port, substitute it in every snippet below.

Default URL:

```text
http://127.0.0.1:56769/mcp
```

## Let your MCP client set it up

Start MiraSpeak and enable the server under **Integrations → MCP**, or let the first prompt walk you through that. Paste the first prompt into your client to have it write the config. After the server is connected, paste the second prompt so the client knows how to use MiraSpeak.

### Set this up for me

```text
Add MiraSpeak as an MCP server in this client. MiraSpeak is a Streamable HTTP server at http://127.0.0.1:56769/mcp (use 127.0.0.1, not localhost). If it isn't listening, enable it in MiraSpeak under Integrations → MCP: select Enable MCP server (localhost only). The default port is 56769.

If this client speaks HTTP or Streamable HTTP natively, point it at that URL:
- Cursor: a mcpServers entry with url in ~/.cursor/mcp.json or .cursor/mcp.json (no type field).
- Claude Code: claude mcp add --transport http miraspeak http://127.0.0.1:56769/mcp. A JSON entry needs "type": "http" (or "streamable-http") plus "url".
- Codex: [mcp_servers.miraspeak] with url in ~/.codex/config.toml, or codex mcp add miraspeak --url http://127.0.0.1:56769/mcp.
- VS Code: in .vscode/mcp.json, a servers entry with "type": "http" and "url". In a workspace-root .mcp.json or Copilot user config (~/.copilot/mcp-config.json), use mcpServers with the same "type": "http" and "url". Don't copy a Cursor entry that has url and no type.

If it only speaks stdio (Claude Desktop's claude_desktop_config.json), bridge with npx -y mcp-remote http://127.0.0.1:56769/mcp --transport http-only. Don't use a "url" entry in Claude Desktop's config file. HTTP on loopback doesn't need --allow-http.

Then check that the MiraSpeak tools appear, and call get_status to verify the connection. Tell me if MCP isn't listening or the port was changed.
```

### Use MiraSpeak

```text
You can control MiraSpeak, a desktop TTS app, through its MCP tools. MiraSpeak must already be running with MCP enabled.

Call get_status before you speak or render audio. If the active engine isn't local (for example Azure), ask me before you spend characters.

Don't invent tool names. Use tools/list if a name fails. Don't try to quit or restart the app, or to install modules; only I can do that from the window. Prefer a short test line the first time you speak.
```

## Cursor

Cursor reads `~/.cursor/mcp.json` (every workspace) or `.cursor/mcp.json` at a project root (that project only). It speaks Streamable HTTP natively:

```json
{
  "mcpServers": {
    "miraspeak": {
      "url": "http://127.0.0.1:56769/mcp"
    }
  }
}
```

Restart Cursor or reload MCP servers, then confirm MiraSpeak is connected and its tools appear.

<!-- vale MiraVoice.Headings = NO -->
## Claude Code
<!-- vale MiraVoice.Headings = YES -->

Claude Code speaks HTTP natively:

```sh
claude mcp add --transport http miraspeak http://127.0.0.1:56769/mcp
```

A JSON entry needs `"type": "http"` (or `"streamable-http"`) plus `"url"`. An entry with a `url` and no `type` is treated as stdio and skipped. Verify with `claude mcp list`.

<!-- vale MiraVoice.Headings = NO -->
## OpenAI Codex CLI
<!-- vale MiraVoice.Headings = YES -->

Codex reads `~/.codex/config.toml` and speaks Streamable HTTP natively:

```toml
[mcp_servers.miraspeak]
url = "http://127.0.0.1:56769/mcp"
```

You can also run `codex mcp add miraspeak --url http://127.0.0.1:56769/mcp`. Verify with `codex mcp list`.

<!-- vale MiraVoice.Headings = NO -->
## Claude Desktop
<!-- vale MiraVoice.Headings = YES -->

Claude Desktop's `claude_desktop_config.json` only launches stdio servers. A `"url"` entry is ignored. Bridge with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) (Node 18+):

```json
{
  "mcpServers": {
    "miraspeak": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://127.0.0.1:56769/mcp",
        "--transport",
        "http-only"
      ]
    }
  }
}
```

| OS | Config path |
| --- | --- |
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

`--transport http-only` skips the SSE probe. HTTP on loopback doesn't need `--allow-http`. Restart Claude Desktop and check **Settings → Developer → MCP Servers**.

Settings → Connectors → Add custom connector is for a public HTTPS URL that Anthropic's cloud can reach. It can't use `127.0.0.1`.

## Other MCP clients

Any client that speaks Streamable HTTP can use `http://127.0.0.1:<port>/mcp`. If it only speaks stdio, use the same `npx -y mcp-remote … --transport http-only` bridge as Claude Desktop.

VS Code examples:

Workspace `.vscode/mcp.json` (editor `servers` format):

```json
{
  "servers": {
    "miraspeak": {
      "type": "http",
      "url": "http://127.0.0.1:56769/mcp"
    }
  }
}
```

Workspace-root `.mcp.json` or Copilot user config (`~/.copilot/mcp-config.json`, `mcpServers` format):

```json
{
  "mcpServers": {
    "miraspeak": {
      "type": "http",
      "url": "http://127.0.0.1:56769/mcp"
    }
  }
}
```

Windsurf, LM Studio, and similar apps take the same URL when they offer an HTTP or Streamable HTTP MCP server field.

## Verify

With MiraSpeak running and MCP listening, ask the client to call `get_status`. A successful reply includes the live engine, voice, queue depth, and translation state. Then try a short `speak` line and `stop_speech` if you need to cut it off.

A connection error usually means MCP is off, MiraSpeak isn't running, the port is wrong, or the URL used `localhost` instead of `127.0.0.1`.

## Tools

`/mcp` is the full tool set for this PC. `/mcp/remote` is the tool set for a tunnel, marked in
the table: it leaves out file reads, deletes, and resets, and a proxied request to `/mcp` gets a
`403`.

<!-- mcp-tools:start (generated by scripts/mcp_tool_table.py) -->
| Tool | Access | Remote | What it does |
| --- | --- | --- | --- |
| `list_presets` | read | yes | List voice presets of the active set, active slot (0 = Manual), and active preset set. |
| `list_preset_sets` | read | yes | List named preset sets and the active set id/name. |
| `switch_preset_set` | writes | yes | Switch the active preset bank. Enters Manual; live voice settings unchanged. |
| `create_preset_set` | writes | yes | Create a named preset set and switch to it. |
| `rename_preset_set` | overwrites or removes | yes | Rename a preset set’s display name (id stays stable). |
| `delete_preset_set` | overwrites or removes | no | Delete a preset set and its presets. This cannot be undone. |
| `switch_preset` | overwrites or removes | yes | Apply a preset slot of the active set, or Manual when slot is 0. A vacant slot fails. |
| `update_preset` | overwrites or removes | yes | Edit an existing preset slot's fields in place (does not enter Manual). |
| `clear_preset` | overwrites or removes | no | Empty a preset slot; its voice is lost (the slot stays). If it was active, enters Manual. |
| `add_preset` | writes | yes | Create an extra preset slot (no hotkey) at the end of the active set and fill it. |
| `delete_preset` | overwrites or removes | no | Delete an extra preset slot (10 and up); later slots move down one number. |
| `speak` | writes, may spend or send | yes | Queue text to speak with the current engine, voice, and preset. |
| `list_personas` | read | yes | List personas (id, name, color, engine, voice, mute, cap_seconds). Changes nothing. |
| `set_persona` | writes | yes | Set the app-wide persona for plain speak. |
| `speak_from_file` | writes, may spend or send | no | Speak a UTF-8 text file by path (no picker). Use it only when the user asks. |
| `stop_speech` | overwrites or removes | yes | Stop the current line, or all queued/playing lines when scope is 'all'. |
| `warm_filler` | writes | yes | Load takes for 60 seconds; same files extend it. Pass only listening to skip the load. |
| `start_filler` | overwrites or removes | yes | Start a filler session from cached takes and return its session_id. |
| `cue_filler` | overwrites or removes | yes | Switch the playing filler to another cue at the next take, and restart its timeout. |
| `stop_filler` | overwrites or removes | yes | Stop the filler session. Playback fades out. |
| `list_filler_cues` | read | yes | List the cues of a persona's filler set, with cached take counts for its voice. |
| `set_speak_by_sentence` | writes | yes | Turn Split sentences on or off (queue one line per sentence when speaking). |
| `set_ui_visible` | writes | yes | Show or hide the main window and quick bar (same as /unhide and /hide). |
| `clear_speak_history` | overwrites or removes | no | Stop playback and wipe the Speak page's line list. This cannot be undone. |
| `set_live_chat_mode` | writes, may spend or send | no | Set the Live Chat mode. tts_to_chat posts each spoken line to the public chat. |
| `set_overlay_position` | writes | yes | Place the quick bar at the bottom or center of the screen. |
| `set_overlay_scale` | writes | yes | Set the quick bar size. The value snaps to a step of 10. |
| `set_live_voice` | overwrites or removes | yes | Change any fields of the live voice in one step (enters Manual). |
| `set_translation` | writes, may spend or send | yes | Turn translation on/off, pick the provider, or the source language. |
| `get_status` | read | yes | Read Manual/preset, the live chips, queue, translation, persona, and speech gates. |
| `list_engines` | read | yes | List TTS engines and whether each is configured (no secrets). |
| `list_voice_options` | read | yes | List one kind of live voice option for an engine, with the current choice. |
| `list_translation_languages` | read | yes | List translation source languages for the active provider and installed Argos pairs. |
| `list_devices` | read | yes | List WASAPI output device names. An empty primary_output means the system default. |
| `refresh_devices` | overwrites or removes | no | Rescan output devices, then list them. This stops playback. |
| `list_themes` | read | yes | List named UI themes and the active theme. |
| `switch_theme` | writes | yes | Switch the active UI theme (applies colors/fonts live). |
| `create_theme` | writes | yes | Create a named theme and switch to it. |
| `rename_theme` | overwrites or removes | yes | Rename a theme’s display name (id stays stable). |
| `delete_theme` | overwrites or removes | no | Delete a user theme and its art. This cannot be undone. |
| `get_appearance` | read | yes | Read active theme colors, fonts, corner image filename, and agent description. |
| `set_appearance` | overwrites or removes | yes | Partial-update the active user theme's tokens. Refuses built-ins; does not enter Manual. |
| `reset_appearance` | overwrites or removes | no | Overwrite the active user theme's tokens with Default (Dark). Cannot be undone. |
| `list_modules` | read | yes | List modules and the tools they expose right now. |
| `call_module` | overwrites or removes, may spend or send | yes | Call a tool on a running module. It runs module code and can speak or spend. |
| `render_audio` | writes, may spend or send | yes | Render text to an audio file without playing it (nothing reaches either output). |
<!-- mcp-tools:end -->

## Errors and gates

Every param has a description in `tools/list`, and every tool has safety hints. A failure sets
`isError`, with text `error: <code>: <message>` and `{code, message, hint}` in
`structuredContent`. Match on `code`, never on `message`.

`get_status.gates` tells a client whether a line can speak now and why not, before it's refused.
`gates.blocked_reason` stays a sentence. Requests with a foreign `Host` or `Origin` get a `403`
on every route.

## Fillers

`gates.filler` is the active persona's filler cache (the live voice when no persona is selected).
`filler.cue` is the set's default cue and `filler.style` its engine style. `start_filler` plays
that cue when no `cue` is passed, or when every cue passed has no takes. It waits the Fillers
lead-in first (default 0.15 seconds, 0 allowed). Pass `lead_in_s` to override that call; a value
outside 0–2 is refused. When Play openers is on and that voice has pinned openers, the first clip
is one of those, then the cue. `list_filler_cues` lists the cue names, which can be renamed on
the Fillers page. `filler.blocked_reason` is a code, or empty when a filler could play.
`filler_playback` is the live session (`null` when none): session id, audible, cue, take, and
milliseconds since it started. audible is false until the first take reaches the player.

## Events

`GET /events` sends `typing_started` and `typing_ended` while you type (no audio). Listening ends
on `listening` false or 35 seconds after the last `listening` true, and either one sends
`typing_ended`. Then `filler_started`, `filler_audible`, and `filler_ended`.

## Personas

`set_persona` sets the app-wide default for lines you type and for `speak` with no `persona`, the
same as the Persona chip, until it's changed. It's not a separate choice per MCP client, and
`""` or `"none"` clears it. `speak(persona_id=)` uses a persona for one line and doesn't change
the default. `get_status.persona` is that default (`null` when none); `engine` and `voice` stay
the live chips. While a persona is active, voice setters fail with `persona_active`. See
[Controls](../architecture/controls.md#personas) for the live voice lock in the window.

## Modules

`call_module` runs a running module's tools, only when it has the MCP tools permission and its
MCP tools switch on the Modules page is on. `granted` in `list_modules` leaves `mcp_tools` out
while that switch is off.

## The speak tool

`speak` and `speak_from_file` with no `source` show an **MCP** badge on the Speak row
(`stop_speech(source="mcp:")` stops those lines). Pass `source` to badge that name instead.
`speak` also takes an optional `persona_id` to speak that one line as that persona without
changing the active persona. A line you type in the app still has no badge.

`speak` says its text as written: slash commands don't run over MCP. Use the matching tool, such
as `switch_preset`, `stop_speech`, `clear_speak_history`, or `set_live_chat_mode`. Only the
person at the PC can quit or restart the app, or add modules.

## The render_audio tool

`render_audio` takes `text`, an optional `slot` (a slot from `list_presets`; default is the live
voice), `format` (`mp3`, `ogg`, or `wav`; default `mp3`), and `confirm` for long text. The line
goes through the normal speaking queue but never plays: it shows on Speak as an **MCP render**
row that ends as **Rendered**, and captions, the latest-line file, Live chat, and the avatar are
skipped. It never translates. The reply holds a relative `path` such as `/renders/<token>.mp3`;
download it with a GET to the same server (and the same headers, if you go through a tunnel)
within 10 minutes. Files live in `%LOCALAPPDATA%\MiraSpeak\renders` and are cleared when MCP
starts or stops. Renders that take longer than 90 seconds are stopped. Cloud engines (Azure)
bill renders like any other line.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| Connection refused | MiraSpeak isn't running, MCP is off, or the port doesn't match |
| Connected, then no tools | The client started before MCP was listening; reload or restart the client |
| Claude Desktop ignores the server | A `"url"` entry was added to `claude_desktop_config.json`; use the `mcp-remote` bridge |
| Hang on first Claude Desktop launch | `npx` is waiting for install confirmation; include `-y` |
| `localhost` works in a browser, fails in the client | The client resolved IPv6 (`::1`); MiraSpeak binds IPv4 only. Use `127.0.0.1` |
| Footer says enabled but not listening | The port is in use. A collision doesn't block app startup; pick another port and save |
