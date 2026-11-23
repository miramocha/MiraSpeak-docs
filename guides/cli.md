# CLI mode

Run MiraSpeak from a terminal, with no window. You set the profile up in the window first. CLI
mode then speaks lines typed in the terminal, and lines sent over MCP, using the saved settings.

Launch commands are also in the MiraSpeak README. This page covers `--cli`, slash commands,
hotkeys, and the tray.

## CLI mode

`--cli` and `--show` can't be used together. Set the profile up in the window first. It creates
no window, tray icon, quick bar, or hotkey. Qt runs offscreen. It does not change engines,
voices, themes, translation, output devices, Live chat, or modules.

The installed app starts in the tray and doesn't offer this flag. Use a source checkout:

```powershell
uv run miraspeak --cli
```

These slash commands work: `/manual`, `/preset1`…, `/preset`, `/presetset`, `/persona`, `/stop`,
`/stopall`, and `/quit`. Any other slash command, including a command alias, prints
`not available in CLI mode`. `/persona` and the `Name:` prefix use the same rules as in the
window; see [Controls](../architecture/controls.md#personas). `/quit`, Ctrl+C, and Ctrl+Break
(SIGTERM on Linux) quit with no confirm.

`//` speaks a leading slash instead of running a command. `/preset2` (and the other slot
commands) can speak text after the command in that voice.

`queued` prints the text as typed. `playing` leads with the same `[style]` badge as the Speak
row when a style hook added one:

```text
queued 3: That's amazing, senpai, I can't believe it!
playing 3: [impressed] That's amazing, senpai, I can't believe it!
done 3
```

A `playing` line with no brackets was spoken as typed.

Saved Live chat still runs. Chat → TTS lines speak unattended.

When MCP is on, both `/mcp` and `/mcp/remote` are served. The tool list is in
[Connect an MCP client](mcp-clients.md). Themes, appearance, and translation languages stay off.
`GET /events` stays. Switching a preset, preset set, or persona writes `settings.toml`, so the
window shows that choice next time. An engine that's not ready can't be fixed from here. Azure
lines still bill; the saved usage limit is the only guard. A second launch of the window while
CLI mode holds the profile shows "MiraSpeak is running in CLI mode."

## Slash commands and hotkeys

The hotkey (default `Ctrl+Shift+Space`) opens the bar over a borderless game. Enter sends.
Shift+Enter sends and keeps the bar open. Select the slash-command field, or type `/` on an
empty line, to search commands:

`/preset1`–`/preset9`, `/manual`, `/stop`, `/stopall`, `/clear`, `/theme`, `/split`, `/livechat`,
`/preset`, `/presetset`, `/engine`, `/file`, `/render`, `/loadmodule`, `/installmodule`.

Up/Down + Enter pick from the popup. Tab from the command field moves to the message (or
abandons a partial command) and doesn't cycle back.

`/theme`, `/split`, `/livechat`, `/preset`, `/presetset`, and `/engine` turn the message field
into a search: type part of a name (a theme, on / off, a Live chat mode, a preset, a preset set,
or an engine) and press Enter. `/engine` enters Manual. `/file` speaks a text file you pick.
`/loadmodule` picks a module folder to run in place. `/installmodule` picks a module .zip and
shows what it can reach before installing. Text after either module command is ignored.

`/render <text>` asks where to save first, then renders the text with the live voice (translated
when Translate is on) to mp3 / ogg / wav / flac without playing it. The Speak row shows a
**Render** badge and ends as **Rendered**.

Ctrl+/ switches engine (Manual) or opens the preset picker. Esc closes the bar. Check **Split
sentences** on Speak or the quick bar (or `/split on`) to speak each sentence as its own line.
**Ctrl+Shift+1–9** applies a voice preset globally (works in-game). Edit slots under **Presets**.
Focus goes back to the game when the bar closes.

**Hide quick bar from recording** and **Hide main window from recording** (tray menu or
**Interface → Quick bar**) keep the quick bar and main window out of OBS Display/Game Capture
independently. Turn either off if you want that surface visible on stream.

The summon hotkey and Stay open are also on **Interface → Quick bar**. See
[Controls](../architecture/controls.md).

## Tray and window

Select the tray icon to open the window. Pages, in order:

| Area | Pages |
| --- | --- |
| Speak | Speak, plus a page for each enabled module |
| Voice engines | Kokoro, VOICEVOX, Azure Speech |
| Voice | Presets, Personas, Commands, Marker aliases |
| Output | Audio devices, Text output |
| Integrations | Live chat, MCP, Modules |
| Interface | Quick bar, Appearance |

The architecture overview has the fuller map, including Translation, Windows, Local TTS, Font
Sets, Effects, and About. See [Architecture overview](../architecture/overview.md#navigation-surface-settings).

Closing the window keeps MiraSpeak running in the tray. Use Quit in the tray menu to exit.
Starting it a second time opens the running window.
