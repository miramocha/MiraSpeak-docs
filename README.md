# MiraSpeak docs

Public architecture and system design for **MiraSpeak**, a desktop TTS tray app for live speaking (VTuber / streamer workflows).

The application source is private. This repository documents how the system is designed—not roadmaps, task boards, or implementation logs.

## Contents

| Doc | Topic |
| --- | --- |
| [Quick start](guides/quick-start.md) | Guide: install MiraSpeak and speak your first line |
| [Architecture overview](architecture/overview.md) | Product shape, hard constraints, layer diagram |
| [Layers](architecture/layers.md) | Responsibilities of each package layer |
| [Line lifecycle](architecture/line-lifecycle.md) | From submit to playback (threads, capture, cache) |
| [Voice engines](architecture/engines.md) | Adapter model, shipped engines, readiness checks |
| [Translation](architecture/translation.md) | Translating lines into the speech language before synthesis |
| [Modules](architecture/modules.md) | Third-party form / web add-ons under Speak |
| [Integrations](architecture/integrations.md) | MCP, Live Chat, Avatar (VMC), captions |
| [Remote MCP via Cloudflare](guides/remote-mcp-cloudflare.md) | Guide: reach the MCP server securely from a cloud agent |
| [Azure Speech + Translator](guides/azure-speech-translator.md) | Guide: create the Azure resources and add keys to MiraSpeak |

## What this repo is not

- Not a changelog or status board  
- Not API reference for unreleased internals  
- Not the place for private source mirrors  
