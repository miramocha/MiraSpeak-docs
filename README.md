# MiraSpeak docs

Public architecture and system design for **MiraSpeak**, a desktop TTS tray app for live speaking (VTuber / streamer workflows).

The application source is private. This repository holds the public architecture, guides, and setup reference: how to run MiraSpeak, connect an MCP client, and configure engines, chat, and modules.

## Contents

| Doc | Topic |
| --- | --- |
| [Quick start](guides/quick-start.md) | Guide: install MiraSpeak and speak your first line |
| [CLI mode](guides/cli.md) | Guide: run from a terminal, slash commands, and the window |
| [Marker tokens](guides/markers.md) | Guide: `[date]`, `[time]`, and marker aliases |
| [Architecture overview](architecture/overview.md) | Product shape, hard constraints, layer diagram |
| [Layers](architecture/layers.md) | Responsibilities of each package layer |
| [Line lifecycle](architecture/line-lifecycle.md) | From submit to playback (threads, capture, cache) |
| [Voice engines](architecture/engines.md) | Adapter model, shipped engines, readiness checks, and in-app setup |
| [Translation](architecture/translation.md) | Translating lines into the speech language before synthesis |
| [Modules](architecture/modules.md) | Third-party modules: pages under Speak, and engine modules that add voice engines |
| [Module API](module-api/README.md) | Reference for writing modules: manifest, author API, form pages, MCP tools, and MiraSpeak callbacks |
| [Integrations](architecture/integrations.md) | MCP, Live chat, Avatar (VMC), and module install |
| [Controls](architecture/controls.md) | Quick bar, global hotkeys, and personas |
| [Outputs](architecture/outputs.md) | Audio devices and text output (captions) |
| [Connect an MCP client](guides/mcp-clients.md) | Guide: enable the server, connect a client, and the generated tool table |
| [Remote MCP via Cloudflare](guides/remote-mcp-cloudflare.md) | Guide: reach the MCP server securely from a cloud agent |
| [Azure Speech + Translator](guides/azure-speech-translator.md) | Guide: create the Azure resources and add keys to MiraSpeak |
| [Discord voice channel](guides/discord.md) | Guide: create your Discord bot and play MiraSpeak in a voice channel |
| [Piper voices](guides/piper.md) | Guide: install the Piper module and download local English voices |
| [Hello world module](guides/module-hello-world.md) | Guide: write, install, and run a first MiraSpeak module |
| [Install a module from GitHub](guides/module-github-install.md) | Guide: publish a module in a repository folder and install it from a link |
| [Build pylock files](guides/module-packages.md) | Guide: lock a module's Python packages for each platform |

## What this repo isn't

- Not a changelog or status board  
- Not API reference for unreleased internals  
- Not the place for private source mirrors
