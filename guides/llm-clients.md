# Connect an LLM client

MiraSpeak's built-in [MCP](https://modelcontextprotocol.io/) server lets a local AI client drive the running app: speak lines, change voices and presets, render audio, and call module tools. This page covers enabling the server and connecting a client. The tool surface is summarized here; later-build behavior is in [Integrations](../architecture/integrations.md#mcp). For a cloud agent that can't reach your PC, see [Remote MCP via Cloudflare](remote-mcp-cloudflare.md).

The snippets below match the current beta (`v0.1.0-beta`). If `tools/list` shows different names than the table here, follow the names the server returns. [Later builds](#later-builds) lists the known renames.

## Enable the server

1. Start MiraSpeak on the same PC as the client.
2. Open **Integrations → MCP**.
3. Select **Enable MCP server (localhost only)**.
4. Leave **Port** at `56769` unless that port is already in use (`mcp_port` in `settings.toml`).

The footer should read `Listening on http://127.0.0.1:<port>/mcp`. MCP is off by default. There is no login on the port: any process on the PC can call every tool while the server is on. Turn it off when you don't need it.

The server is Streamable HTTP, bound to `127.0.0.1` only. There is no stdio transport. Use `127.0.0.1` in client config, not `localhost` (which can resolve to IPv6).

Default URL:

```text
http://127.0.0.1:56769/mcp
```

If you changed the port, substitute it in every snippet below.

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

## Claude Code

Claude Code speaks HTTP natively:

```sh
claude mcp add --transport http miraspeak http://127.0.0.1:56769/mcp
```

A JSON entry needs `"type": "http"` (or `"streamable-http"`) plus `"url"`. An entry with a `url` and no `type` is treated as stdio and skipped. Verify with `claude mcp list`.

## OpenAI Codex CLI

Codex reads `~/.codex/config.toml` and speaks Streamable HTTP natively:

```toml
[mcp_servers.miraspeak]
url = "http://127.0.0.1:56769/mcp"
```

You can also run `codex mcp add miraspeak --url http://127.0.0.1:56769/mcp`. Verify with `codex mcp list`.

## Claude Desktop

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

Workspace-root `.mcp.json` or Copilot user config (`mcpServers` format):

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

With MiraSpeak running and MCP listening, ask the client to call `get_status`. A successful reply includes the live engine, voice, queue depth, and translation state. Then try a short `speak` line and `stop` if you need to cut it off.

A connection error usually means MCP is off, MiraSpeak isn't running, the port is wrong, or the URL used `localhost` instead of `127.0.0.1`.

## Tools (current beta)

`tools/list` is the source of truth. Names below are the handlers the beta server registers, including `call_module` and `render`.

| Group | Tools |
| --- | --- |
| Speak | `speak`, `speak_from_file`, `stop`, `set_speak_by_sentence`, `render` |
| Status and catalogs | `get_status`, `list_engines`, `list_models`, `list_voices`, `list_voice_banks`, `list_voice_characters`, `list_styles`, `list_paces`, `list_markers`, `list_speech_languages`, `list_translation_languages`, `list_devices` |
| Live voice | `set_engine`, `set_model`, `set_voice`, `set_voice_bank`, `set_voice_character`, `set_styles`, `set_pace`, `set_speech_language`, `set_pitch`, `enter_manual` |
| Presets | `list_presets`, `switch_preset`, `update_preset`, `clear_preset`, `add_preset`, `delete_preset`, `list_preset_sets`, `switch_preset_set`, `create_preset_set`, `rename_preset_set`, `delete_preset_set` |
| Translation | `set_translation` (target language is `set_speech_language`; `get_status.translation` reports the state) |
| Themes | `list_themes`, `switch_theme`, `create_theme`, `rename_theme`, `delete_theme`, `get_appearance`, `set_appearance`, `reset_appearance` |
| Modules | `list_modules`, `call_module` |
| Window | `set_ui_visible` |

Useful arguments and replies:

- `speak` takes `text`. It returns `{ok, queue_depth}` when the line is queued.
- `stop` takes `scope`: `current` (default) or `all`.
- `speak_from_file` takes a UTF-8 `path`. Over about 2,000 characters it returns `reason=needs_confirm`; ask the user, then call again with `confirm=true`.
- `render` takes `text`, optional `preset` (a slot from `list_presets`), and optional `format`. It writes an audio file and does not play it. GET the returned `path` from this same server within `expires_in_s` (10 minutes).
- `call_module` takes `module_id`, `tool`, and optional `arguments` from `list_modules`.
- Live voice setters enter Manual. `update_preset` and the theme / appearance tools do not. `set_translation` does not enter Manual.
- Commands that add modules, or that would open a file dialog, are refused. Call `speak_from_file` or `render` instead. Only the person at the PC can add modules.

Call `get_status` before `speak` or `render`, and ask first when the active engine isn't local (for example Azure).

## Copyable prompts

Paste the long prompt into the client so it knows how to use MiraSpeak. Paste the short prompt when you want the client to write the config for you.

### Use MiraSpeak

```text
You can control MiraSpeak, a desktop TTS app, through its MCP tools. MiraSpeak must already be running with MCP enabled.

Call get_status before speak or render. If the active engine is not local (for example Azure), ask me before you spend characters. After you queue a line, you can stop it with stop (scope "current" or "all").

speak({text}) queues a line with the current engine, voice, and preset. It returns ok and queue_depth.

speak_from_file({path}) reads a UTF-8 file. If the result is needs_confirm, ask me, then call again with confirm=true.

render({text}, optional preset, optional format) writes an audio file and does not play it. GET the returned path from the same MCP server within expires_in_s.

Live voice tools (set_engine, set_model, set_voice, set_voice_bank, set_voice_character, set_styles, set_pace, set_speech_language, set_pitch, enter_manual) switch to Manual. update_preset and theme/appearance tools do not. set_translation turns translation on or off and picks the provider and source; set_speech_language is the target. get_status.translation reports the state.

list_presets / switch_preset and list_preset_sets / switch_preset_set change saved voices. list_themes / switch_theme and get_appearance / set_appearance change the UI. list_modules then call_module({module_id, tool, arguments}) runs a module tool.

Do not invent tool names. Use tools/list if a name fails. Do not try to quit or restart the app, or to install modules; only I can do that from the window. Prefer a short test line the first time you speak.
```

### Set this up for me

```text
Add MiraSpeak as an MCP server in this client. MiraSpeak is already a Streamable HTTP server at http://127.0.0.1:56769/mcp (use 127.0.0.1, not localhost). I will turn the server on in MiraSpeak under Integrations → MCP if it isn't listening.

If this client speaks HTTP or Streamable HTTP natively, point it at that URL (Claude Code: `claude mcp add --transport http miraspeak http://127.0.0.1:56769/mcp`; Codex: `[mcp_servers.miraspeak]` with `url` in ~/.codex/config.toml; Cursor: `url` in ~/.cursor/mcp.json or .cursor/mcp.json). If it only speaks stdio (Claude Desktop's claude_desktop_config.json), bridge with `npx -y mcp-remote http://127.0.0.1:56769/mcp --transport http-only`. Don't use a "url" entry in Claude Desktop's config file.

Then check that the MiraSpeak tools appear, and call get_status to verify the connection. Tell me if MCP isn't listening or the port was changed.
```

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| Connection refused | MiraSpeak isn't running, MCP is off, or the port doesn't match |
| Connected, then no tools | The client started before MCP was listening; reload or restart the client |
| Claude Desktop ignores the server | A `"url"` entry was added to `claude_desktop_config.json`; use the `mcp-remote` bridge |
| Hang on first Claude Desktop launch | `npx` is waiting for install confirmation; include `-y` |
| `localhost` works in a browser, fails in the client | The client resolved IPv6 (`::1`); MiraSpeak binds IPv4 only — use `127.0.0.1` |
| Footer says enabled but not listening | The port is in use. A collision doesn't block app startup; pick another port and save |
| Tool name not found (`stop_speech`, `render_audio`, `list_voice_options`) | You're on a later build than this table; use `tools/list` and see [Later builds](#later-builds) |

## Later builds

Builds after `v0.1.0-beta` keep the same transport, bind, default port, and `/mcp` path, and still have no login on the port. They also add:

| Area | Change |
| --- | --- |
| Remote endpoint | `/mcp/remote` serves a reduced tool set for a tunneled client. A proxied request to `/mcp` is refused. See [Remote MCP via Cloudflare](remote-mcp-cloudflare.md). |
| Host guard | Every route answers 403 to a foreign `Host` or `Origin`. A tunnel must rewrite `Host` to `localhost:<port>`. |
| Events | `GET /events` is a read-only SSE stream of line events. |
| Renames | `stop` becomes `stop_speech` (adds `line_ids` and `source`). `render` becomes `render_audio`. |
| Merged tools | Live voice setters become `set_live_voice`. Catalog `list_models`, `list_voices`, `list_voice_banks`, `list_voice_characters`, `list_styles`, `list_paces`, `list_markers`, and `list_speech_languages` become `list_voice_options`. `enter_manual` is dropped. |
| New tools | `list_personas`, `set_persona`, `clear_speak_history`, `set_live_chat_mode`, `set_overlay_position`, `set_overlay_scale`, `refresh_devices`. |
| Speak | `speak` and `speak_from_file` return `line_ids`, take an optional `source` tag, and (for `speak`) an optional `persona_id`. `speak_from_file` is limited to folders the user lists on the MCP page. |

Those details live in [Integrations](../architecture/integrations.md#mcp). The copyable prompts above follow the beta names; if `tools/list` already shows the later names, use those instead.
