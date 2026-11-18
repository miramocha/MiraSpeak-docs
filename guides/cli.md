# CLI mode

This feature isn't in the current MiraSpeak build yet.

Run MiraSpeak from a terminal, with no window. You set the profile up in the window first. CLI mode then speaks lines you type, and lines an agent sends over MCP, using those saved settings.

CLI mode runs from a source checkout:

```powershell
uv run miraspeak --cli
```

`--cli` and `--show` can't be used together. The installed app starts in the tray and doesn't offer this flag.

There is no window, tray icon, quick bar, or hotkey. A beta build uses the sign-in you already saved. With no valid sign-in it prints `run once with the window to sign in` and exits.

## Lines you type

Each line is spoken the same way as a line in the Speak box. `//` speaks a leading slash.

| You type | What happens |
| --- | --- |
| `/persona Natsuki` | Later lines speak as that persona. The name or the id works. |
| `/persona none` | Clears the persona. |
| `Natsuki: hello` | Speaks that one line as Natsuki. |
| `Say \: this` | Speaks a colon. |
| `/preset2` | Switches to preset 2. Speaks text after the command in that voice. |
| `/manual` | Switches to Manual. |
| `/preset Name` | Switches to a preset by name. |
| `/presetset Name` | Switches preset set. |
| `/stop` | Stops the line that's playing. |
| `/stopall` | Stops every line. |
| `/quit` | Quits, with no confirm. |

Any other slash command, including a command alias, prints `not available in CLI mode`.

Ctrl+C quits. On Windows, Ctrl+Break quits too. On Linux, SIGTERM quits.

`queued` prints the text you typed. `playing` leads with the same `[style]` badge as the Speak row when a module added one:

```text
queued 3: That's amazing, senpai, I can't believe it!
playing 3: [impressed] That's amazing, senpai, I can't believe it!
done 3
```

A `playing` line with no brackets was spoken as typed.

Saved Live chat still runs. Chat → TTS lines speak while CLI mode is up.

## MCP

When MCP is turned on in the window, both `/mcp` and `/mcp/remote` serve the tools listed in [Connect an MCP client](mcp-clients.md#tools), including `get_status`, `speak`, `stop`, `render`, preset tools, and `call_module`. Themes, appearance, and translation languages stay in the window. `render` returns a path to download from this same server. The rest of the MCP behavior is in [Integrations](../architecture/integrations.md#mcp).

## What stays in the window

CLI mode doesn't change engines, voices, themes, translation, output devices, Live chat, or modules. Do that in the window, then start CLI mode again.

Switching a preset, a preset set, or a persona writes `settings.toml`, so the window shows that choice the next time it opens.

A second launch of the window, while CLI mode holds that profile, shows **MiraSpeak is running in CLI mode.** The CLI process keeps running.

Azure lines still bill. The saved usage limit is the only guard.
