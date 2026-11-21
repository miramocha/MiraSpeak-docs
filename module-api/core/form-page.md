# Form pages

A module's page is plain data. `Module.page()` returns a `ui.Page`, and the host app draws it with its own widgets, so a module supplies no widgets, styles, or colors. Code: `cetimodule.ui`.

```python
from miraspeak.modules.api import ui

ui.Page(
    intro="Say a greeting with the voice bar below.",
    sections=(
        ui.Section("Greeting", (
            ui.Text("greeting", "Greeting", default="Hello, chat."),
            ui.Button("say", "Say it", speaks=True),
            ui.Status("status", "Press Say it."),
        )),
    ),
)
```

## Page and sections

| Type | Fields |
| --- | --- |
| `ui.Page` | `intro` (text above the sections), `sections` |
| `ui.Section` | `title`, `fields`, `bar` (default `False`) |

`bar=True` draws the section as a pinned toolbar at the top of the page, not as a titled form block. Non-Choice fields sit left to right on one row; `ui.Choice` fields sit on a second row and expand. A `ui.Button` with `header=True` goes to the page's title row instead of the bar.

Every field has a `key`, unique on the page. `ctx.values`, `ctx.ui`, and the callbacks name fields by `key`.

## Value fields

These fields hold a value that the app saves and restores in `ctx.values`.

| Type | Value | Options |
| --- | --- | --- |
| `ui.Text` | `str` | `label`, `default`, `placeholder`, `mono` (a fixed-width readout for a URL or host) |
| `ui.TextArea` | `str` | `label`, `default`, `placeholder` |
| `ui.Toggle` | `bool` | `label`, `default` |
| `ui.Number` | `int` when `decimals` is 0, else `float` | `label`, `default` (0), `minimum` (0), `maximum` (100), `step` (1), `decimals` (0), `suffix` |
| `ui.Slider` | Same as `Number` | Same as `Number`, plus `visible` (first paint). The value is saved when the slider is released. |
| `ui.Choice` | The option id | `label`, `options` (`(id, label)` pairs), `default`, `enabled` (first paint) |
| `ui.File` | The path | `label`, `extensions`, `default`, `inline`. **Browse** sits on the pinned title row; `inline=True` puts it in the field's row, followed by any buttons right after the field. |

## Other fields

| Type | Purpose | Options |
| --- | --- | --- |
| `ui.Button` | Calls `on_action(key)` | `label`, `enabled`, `header` (put it on the pinned title row), `speaks` |
| `ui.Status` | A read-only hint line, set with `ctx.ui.set_status` | `text`, `tone` (`ui.TONE_NEUTRAL` or `ui.TONE_WARNING`) |
| `ui.Progress` | A playback strip, set with `ctx.ui.set_progress` | `label` |
| `ui.Meters` | Read-only 0–1 bars, set with `ctx.ui.set_meters`. Not saved. | `label` |
| `ui.Rows` | A text list, set with `ctx.ui.set_rows`. Not saved. | `label`, `row_actions` (a button per row), `click` (action id for selecting a line), `when_empty`, `speaks` |
| `ui.Secret` | A masked field kept in the OS credential store ([Secrets](author-api.md#secrets)) | `label`, `placeholder` |
| `ui.LineBox` | A line pinned under the page: a text box, one `Choice`, and a send button. The text isn't saved. | `agent` (the key of the `Choice`), `action` (default `"send"`), `button` (default `"Send"`), `placeholder`, `status` (the key of a `Status`) |

A row action or a selected line calls `on_action(action, row=index)`. Enter or the send button on a `ui.LineBox` calls `on_action(action)`, and `ctx.line(key)` returns the text.

### Empty lists

`ui.Empty` is what a `ui.Rows` list shows while it has no rows. Set it with `Rows.when_empty`, or later with `ctx.ui.set_empty`.

| Field | Meaning |
| --- | --- |
| `title` | Required heading |
| `body` | Text under the title |
| `action` | Adds a button. The key of a `ui.File` field opens that field's Browse; any other id calls `on_action(action)`. A `File` key wins over an action id with the same name. |
| `action_label` | Button text. Blank shows "Browse…" for a `File` key, else the id. |
| `art` | One of `ui.ART_NAMES` (`book`, `list`, `file`), or blank. Art the app doesn't ship yet shows text only. |
| `speaks` | The action starts speech |

### Speech controls

`speaks=True` on `ui.Button`, `ui.Rows`, or `ui.Empty` marks a control that starts speech. The app keeps it off while the speaking engine isn't ready. Leave it off for a button that also stops, such as a Start/Stop toggle.
