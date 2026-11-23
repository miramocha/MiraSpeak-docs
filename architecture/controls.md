# Controls: Quick bar, hotkeys, and personas

The quick bar is a floating composer summoned with a global hotkey. Hotkeys are registered with
the OS (`RegisterHotKey`) without low-level keyboard hooks. Global hotkeys are Windows only.
`Ctrl+Shift+1` through `Ctrl+Shift+9` apply presets 1 to 9. `Ctrl+Alt+1` through `Ctrl+Alt+9`
activate the first 9 personas. Those chords are fixed.

Set the summon hotkey and Stay open on **Interface → Quick bar**. The slash-command catalog and
the hide-from-recording checkboxes (**Hide main window from recording** and **Hide quick bar from
recording**) are in [CLI mode](../guides/cli.md#slash-commands-and-hotkeys).

## Personas

Personas are named voices that sit beside presets. Each one owns its own engine and voice, a
color, and optional corner art. Make and edit them under **Voice → Personas**. New personas copy
the live voice. **Copy from preset** copies a preset's voice once.

| Action | What it does |
| --- | --- |
| One line | Start a line with a persona's name and a colon, for example `Natsuki: Hello!`, to speak only that line as that persona. The name must match exactly. Type `\:` for a literal colon. |
| Switch | **Ctrl+Alt+1–9** makes one of the first nine personas active (works in-game). Choosing a preset or Manual clears the active persona. In the terminal, `/persona Natsuki` speaks later lines as that persona. The name or the id works, and `/persona none` clears it. |
| Live voice lock | While a persona is active, typed lines speak with its voice, so live voice edits (engine, voice, style, and similar) are refused until you clear it. |
| History | A persona line shows the persona's name in its color on the Speak row. |
