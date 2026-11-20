# Module API

Reference for writing a module: a folder with a `module.toml` and a Python entry class that a Ceti app loads and runs. MiraSpeak is the first app that hosts modules.

The API has two layers:

1. CetiModule is the shared, app-independent part: the manifest, the `Module` base class, the context (`ctx`), form pages, and MCP tools. Every app that hosts modules uses the same objects.
2. The host app adds its own permissions, context methods, and callbacks. For MiraSpeak, that's speech, line events, playback, and voice engines.

A MiraSpeak module imports everything from `miraspeak.modules.api`, which re-exports the CetiModule names as the same objects.

## Pages

| Doc | Topic |
| --- | --- |
| [Module manifest](core/manifest.md) | `module.toml` fields, `requires` pins, and `pylock` files |
| [Author API](core/author-api.md) | The `Module` callbacks, `ctx`, timers, secrets, data files, errors, and shared permissions |
| [Form pages](core/form-page.md) | The `ui` field types a page is built from |
| [MCP tools](core/mcp-tools.md) | Publishing module tools and instructions to the app's MCP server |
| [For host apps (may change)](core/host-apps.md) | `cetimodule.host`, for apps that load modules |
| [MiraSpeak module API](miraspeak.md) | `miraspeak.modules.api`: permissions, speech, hooks, and engine modules |
| [Hello world module](../guides/module-hello-world.md) | Guide: write, install, and run a first MiraSpeak module |
| [Install a module from GitHub](../guides/module-github-install.md) | Guide: publish a module in a repository folder and install it from a link |
| [Build pylock files](../guides/module-packages.md) | Guide: lock a module's Python packages for each platform |

Only the author API is stable. A breaking change to it raises the host app's module API version (MiraSpeak `API_VERSION`, now 1), and modules built for the old version show *Needs update* and don't load.

<!-- When CetiModule's own docs/ folder is public, core/ moves there as is. Links into core/ from the MiraSpeak pages then point at the CetiModule docs. -->
