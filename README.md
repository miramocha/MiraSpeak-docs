# MiraSpeak docs

Public architecture and system design for **MiraSpeak**, a desktop TTS tray app for live speaking (VTuber / streamer workflows).

The application source is private. This repository documents how the system is designed—not roadmaps, task boards, or implementation logs.

## Contents

| Doc | Topic |
| --- | --- |
| [Architecture overview](architecture/overview.md) | Product shape, hard constraints, layer diagram |
| [Layers](architecture/layers.md) | Responsibilities of each package layer |
| [Line lifecycle](architecture/line-lifecycle.md) | From submit to playback (threads, capture, cache) |
| [Voice engines](architecture/engines.md) | Adapter model and shipped engines |
| [Modules](architecture/modules.md) | Third-party form / web add-ons under Speak |
| [Integrations](architecture/integrations.md) | MCP, Live Chat, Avatar (VMC), captions |

## What this repo is not

- Not a changelog or status board  
- Not API reference for unreleased internals  
- Not the place for private source mirrors  
