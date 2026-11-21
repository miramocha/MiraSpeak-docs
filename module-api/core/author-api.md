# Author API

A module's entry class subclasses `Module` and overrides the callbacks it needs. CetiModule passes it a context, `ctx`, for everything it does in the app. Code: `cetimodule.api` (`Module`, `ModuleContext`, the handles, the errors) and `cetimodule.context`.

A MiraSpeak module imports these names from `miraspeak.modules.api` instead. They're the same objects, plus MiraSpeak's own; see the [MiraSpeak module API](../miraspeak.md).

## Lifecycle

1. The host imports the `entry` class from [`module.toml`](manifest.md) and creates it.
2. `page()` returns the form page, or `None` for no page. The host calls it once, before `on_start`.
3. `on_start(ctx)` runs. Keep `ctx` for later calls.
4. While the module runs, the host delivers events: `on_action`, `on_value`, `on_secret`, timer callbacks, MCP tool calls, and the host app's own callbacks.
5. `on_stop()` runs when the module is disabled or the app quits. Cancel timers and close what the module opened.

Every callback runs on the module's own thread, one at a time. A module never calls its own callbacks to reach the host; it calls `ctx`.

## Callbacks

| Callback | When |
| --- | --- |
| `page() -> ui.Page \| None` | Once, before `on_start`. Default `None`. |
| `on_start(ctx)` | The module starts. |
| `on_stop()` | The module stops. |
| `on_action(action, row=None)` | A `ui.Button` is selected, a `ui.LineBox` is sent, or a `ui.Rows` row action or line is selected. `row` is the row index for a row action, else `None`. |
| `on_value(key, value)` | A user edit of a value field on the page. A module's own `ctx.ui.set()` does not fire it. |
| `on_secret(key)` | The user saved or cleared the `ui.Secret` field `key`. Read it with `ctx.secrets.get(key)`. |
| `mcp_instructions() -> str` | Text for the app's MCP server; see [MCP tools](mcp-tools.md). Default `""`. |
| `mcp_tools() -> Mapping[str, Callable]` | Tools for the app's MCP server. Default `{}`. |

## Context

`ctx` is a `ModuleContext`. A host app's context extends it.

| Member | Purpose |
| --- | --- |
| `ctx.module_id` | The manifest `id` |
| `ctx.log` | A `logging.Logger` for this module |
| `ctx.values` | Current page field values, read-only. Restored when the module starts. |
| `ctx.ui` | Updates to the module's own page ([Page updates](#page-updates)) |
| `ctx.secrets` | The module's secrets ([Secrets](#secrets)) |
| `ctx.data` | The module's own text files ([Data files](#data-files)) |
| `ctx.open_url(url)` | Opens a plain `https` link in the user's browser. Needs `network`. |
| `ctx.call_later(seconds, fn)` | Runs `fn` once after `seconds`. Returns a `Timer`. |
| `ctx.every(seconds, fn)` | Runs `fn` every `seconds`. Returns a `Timer`. |
| `ctx.line(key)` | The text last sent from the `ui.LineBox` `key`, or `""` before the first send |

### Values

`ctx.values` maps each value field's `key` to its value. The type follows the field: `str` for `Text`, `TextArea`, `Choice` (the option id), and `File` (the path); `bool` for `Toggle`; `int` for `Number` and `Slider` with `decimals = 0`, else `float`. The app saves values between runs. Buttons, status lines, rows, meters, line boxes, and secrets aren't values.

### Page updates

`ctx.ui` changes the module's own page. The app applies each change on its GUI thread.

| Method | Effect |
| --- | --- |
| `set(key, value)` | Sets a value field |
| `set_status(key, text, tone=ui.TONE_NEUTRAL)` | Sets a `ui.Status` line. `tone` is `ui.TONE_NEUTRAL` or `ui.TONE_WARNING`. |
| `set_progress(key, elapsed_s, total_s)` | Moves a `ui.Progress` strip. Both times are seconds. |
| `set_meters(key, values)` | Fills a `ui.Meters` block: one bar per name, each clamped to 0–1. Not saved. About 30 calls a second is fine. |
| `set_enabled(key, enabled)` | Turns a field on or off |
| `set_visible(key, visible)` | Shows or hides a field's whole row, label included |
| `set_label(key, text)` | Changes a button's caption |
| `set_line(key, text)` | Replaces the text of a `ui.LineBox`. Not saved. |
| `set_rows(key, rows)` | Replaces a `ui.Rows` list. Not saved. |
| `highlight_row(key, index)` | Marks one row of a `ui.Rows` list. `None` clears the mark. |
| `mark_rows(key, indexes)` | Sets which rows show the cached label. Separate from the highlight. |
| `set_empty(key, empty)` | Sets what an empty `ui.Rows` list shows (`ui.Empty`). `None` shows nothing. |
| `set_options(key, options)` | Replaces the `(id, label)` items of a `ui.Choice`. Not saved; the selected id is. |

The user's edits win. In MiraSpeak, a module's `set` that hasn't seen the user's latest edit to that field is dropped.

### Secrets

A `ui.Secret` field keeps its text in the OS credential store, never in the saved values. The page is the only writer.

| Method | Returns |
| --- | --- |
| `get(key)` | The saved text, or `None` when it's unset or the credential store isn't available |
| `copy(field, dest)` | Copies the `ui.Secret` `field` to the entry `dest`, then clears the field. `False` when the field is empty; the stored `dest` stays as it is. `dest` is a short name, not a field on the page. |
| `clear(key)` | Deletes a saved secret. A page field's box refreshes. |

### Data files

`ctx.data` reads and writes text files in the module's own folder, `module_data/<id>/`. The folder is removed with the module, unless the user keeps its data.

| Method | Returns |
| --- | --- |
| `read_text(name)` | The file's text, or `None` before it exists |
| `write_text(name, text)` | Replaces one file atomically. `name` is one file name; the text is at most 256 KB. |

Writes run on the module's thread. A module with no data folder gets `ModuleError` from both methods.

### Timers

`call_later` and `every` return a `Timer`. `timer.cancel()` stops it. The callback runs on the module's thread like any other callback. A repeating interval shorter than 0.05 seconds (`MIN_TIMER_INTERVAL_S`) is raised to 0.05 seconds, not refused. A one-shot `call_later` delay can be shorter, including 0. When the module stops, the event loop exits before `on_stop` and remaining timers do not fire.

`ctx.call_later(0, fn)` is how a thread the module started hands work back to the module thread.

### Open a link

`ctx.open_url(url)` opens an `https` link in the user's default browser. Anything else raises `UrlRefusedError`: `http`, `file`, a user name in the link, spaces, or no host. Without the `network` grant it raises `PermissionDeniedError`. Pages have no clickable text, so a link is a `ui.Button` whose `on_action` calls `open_url`.

## Errors

Every error a module can catch subclasses `ModuleError`.

| Error | Raised when |
| --- | --- |
| `ModuleError` | Base class. A host app's own refusals subclass it too. |
| `PermissionDeniedError` | The module wasn't granted the permission the call needs |
| `UrlRefusedError` | `open_url` refused the address |
| `ModuleStoppedError` | The module was disabled or the app is quitting |
| `HostBusyError` | The app didn't answer within 2 seconds (`HOST_CALL_TIMEOUT_S`). Try again later. |

A call that timed out never runs later.

## Limits

- **Errors.** An exception that escapes a callback on the module thread counts as an error: page callbacks, timers, host-delivered hooks, and MCP tool calls. After 3 errors in 60 seconds (`ERROR_LIMIT`, `ERROR_WINDOW_S`), the module stops. An exception in `on_start` fails the start immediately. An exception in `on_stop` is logged and does not count.
- **Stopping.** After the event loop exits, `on_stop` runs. The app waits up to 2 seconds (`STOP_BUDGET_S`) for the module thread to finish. If `on_stop` runs past that, the wait ends and the thread stays running as a daemon.
- **Event queue.** The module's queue holds up to 256 events (`MAX_EVENTS`). When the queue is full, the oldest droppable event is dropped first. In MiraSpeak those are `on_line`, `on_chat_message`, `on_audio`, `on_mouth`, `on_voices_changed`, and a waiting `on_playback_signal` frame. If none are droppable, the oldest non-MCP event is dropped. A queued MCP call is never dropped; if the queue holds only MCP calls, a new event is refused (an MCP call then returns `unavailable`).
