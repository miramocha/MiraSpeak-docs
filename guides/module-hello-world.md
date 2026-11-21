# Hello world module

This guide builds a MiraSpeak module with one text field and one button that speaks the text with the live voice. The API it uses is in the [MiraSpeak module API](../module-api/miraspeak.md).

## 1. Create the folder

A module is a folder with a `module.toml` at its top and the Python file it names:

```text
hello/
  module.toml
  main.py
```

## 2. Write the manifest

`module.toml`:

```toml
id = "hello"
name = "Hello"
version = "0.1.0"
api = 1
entry = "main:HelloModule"
description = "Say a greeting with the live voice."
author = "<your-name>"
permissions = ["speak"]
```

`entry = "main:HelloModule"` points at the class `HelloModule` in `main.py`. `api = 1` matches MiraSpeak's module API version. The module asks only for `speak`. Every key is in [Module manifest](../module-api/core/manifest.md).

## 3. Write the module

`main.py`:

```python
"""Say a greeting with the live voice."""

from __future__ import annotations

from miraspeak.modules.api import LIVE, Module, ModuleContext, ModuleError, ui


class HelloModule(Module):
    def page(self) -> ui.Page:
        return ui.Page(
            intro="Say a greeting with the voice bar below.",
            sections=(
                ui.Section(
                    "Greeting",
                    (
                        ui.Text("greeting", "Greeting", default="Hello, chat."),
                        ui.Button("say", "Say it", speaks=True),
                        ui.Status("status", "Press Say it."),
                    ),
                ),
            ),
        )

    def on_start(self, ctx: ModuleContext) -> None:
        self.ctx = ctx

    def on_action(self, action: str, row: int | None = None) -> None:
        if action != "say":
            return
        text = str(self.ctx.values.get("greeting", "")).strip()
        if not text:
            self.ctx.ui.set_status("status", "Write a greeting first.", tone=ui.TONE_WARNING)
            return
        try:
            self.ctx.speak(text, voice=LIVE)
        except ModuleError as e:
            self.ctx.ui.set_status("status", str(e), tone=ui.TONE_WARNING)
            return
        self.ctx.ui.set_status("status", "Queued.")
```

- `page()` returns the form: a text field, a button, and a status line. `speaks=True` keeps the button off while the speaking engine isn't ready.
- `on_start` keeps `ctx` for later calls.
- `on_action` runs on the module's thread when the button is selected. It reads the field from `ctx.values` and queues the line with `ctx.speak`.
- `ModuleError` covers the refusals `speak` can raise, such as `SpeakRefusedError` for a full queue. The message goes to the status line.

## 4. Install it

On **Integrations → Modules**, select **Install from .zip…**, or **Load from folder…** to run a local folder without zipping (for module authors). You can also push it to a public GitHub repository and [install it from a link](module-github-install.md). The manifest can sit at the zip root or inside one top-level folder.

The consent screen lists **speak**. Approve it, and the **Hello** page appears under Speak with the voice bar pinned at the bottom.

## 5. Try it

Select **Say it**. MiraSpeak queues "Hello, chat." with the live voice, and the status line shows "Queued."

## Next steps

- Add a schedule with `ctx.every`; the [Timer module](https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/timer) is a complete example.
- Add tools for agents with [MCP tools](../module-api/core/mcp-tools.md).
- Add Python packages with `requires` and [pylock files](module-packages.md).
