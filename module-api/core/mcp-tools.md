# MCP tools

A module can add tools and instructions to its host app's MCP server, so an agent connected to the app can call it. The module needs the `mcp_tools` permission. Code: `cetimodule.mcp_surface`.

```python
class HelloModule(Module):
    def mcp_instructions(self) -> str:
        return "To say a greeting, call the hello module's say tool with text."

    def mcp_tools(self) -> dict:
        return {"say": self.mcp_say}

    def mcp_say(self, text: str, repeat: int = 1) -> dict:
        """Say text with the live voice, up to 3 times."""
        ...
        return {"ok": True}
```

## Instructions

`mcp_instructions()` returns text that tells an agent when to use the module. The text is trimmed and holds at most 2,000 characters (`MCP_INSTRUCTIONS_MAX`). Empty adds nothing.

## Tools

`mcp_tools()` maps each tool name to a callable. The host reads the mapping when the module loads.

- **Count.** At most 16 tools (`MCP_TOOL_LIMIT`).
- **Name.** 1–32 characters: a lowercase letter, then lowercase letters, digits, or underscores.
- **Parameters.** Each parameter is a plain or keyword-only parameter annotated `str`, `int`, `float`, or `bool`, which map to JSON `string`, `integer`, `number`, and `boolean`. No `*args` or `**kwargs`.
- **Defaults.** A parameter with a default is optional. The default must match its type, and a `float` default must be finite.
- **Description.** The callable's docstring.
- **Result.** A JSON object: a `dict` with `str` keys and JSON values (text, numbers, `true` / `false`, `None`, lists, and objects). Floats must be finite.

A mapping that breaks one of these rules fails the module load with a message naming the tool, such as `MCP tool 'say' parameter 'text' needs a str, int, float, or bool annotation.`

## Calls

A tool call runs on the module's thread, like a callback, and has 10 seconds (`MCP_CALL_TIMEOUT_S`). The app's thread doesn't wait on it.

In MiraSpeak, agents find module tools with the `list_modules` MCP tool and run them with `call_module`. A module's tools and instructions appear only while it's running and holds the `mcp_tools` grant, and only while the MCP tools switch on the Modules page is on.

`call_module` takes `module_id`, `tool`, and optional `arguments` (an object that matches the tool's schema). A timeout returns `code` `timeout`. A tool that doesn't return a JSON object, or that raises, returns `code` `module_error`. Other codes an agent can see: `unknown_module`, `unknown_tool`, `unavailable` (not running, busy, or stopped), `permission_denied`, `invalid_arguments`, and `module_refused` when the tool returns `{ok: false}`. A module may raise `McpToolError`, but the runtime still reports it as `module_error` and does not keep the original code.
