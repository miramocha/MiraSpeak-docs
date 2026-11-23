# Quick start

Install MiraSpeak, get a free local voice, and hear your first line. It takes about 10 minutes, most of it the one-time voice download.

- You don't need an account, an API key, or a virtual audio cable to get started.
- Streaming setup (OBS), avatar mouth movement (VMC), and cloud voices are optional. See [Next steps](#7-next-steps).

## 1. What you need

- Windows 10 or 11 (x64), or a 64-bit Linux desktop with PulseAudio or PipeWire.
- Free disk space for the Kokoro voice models (a download of about 300 MB).

## 2. Install and start MiraSpeak

<!-- TODO: replace with the release URL once the first public build is published. -->

1. Download `MiraSpeak-<version>-windows-x64.zip` from <download link>.
2. Extract the zip anywhere you like, for example `C:\Apps\MiraSpeak`. Keep the whole `MiraSpeak` folder together.
3. Run `MiraSpeak.exe`. The build isn't code-signed yet, so Windows SmartScreen may warn you: select **More info**, then **Run anyway**.

MiraSpeak starts in the system tray (check the **^** overflow next to the clock if you don't see the icon). Select the icon to open the window. Closing the window keeps MiraSpeak running in the tray. To exit, right-click the icon and choose **Quit**.

**On Linux:** download `MiraSpeak-<version>-linux-x86_64` from the same release, extract it, and run `MiraSpeak` from the folder. Your desktop needs a system tray (most do; on GNOME, install the AppIndicator extension). Global hotkeys (the quick bar shortcut and **Ctrl+Shift+1–9**) are Windows-only for now, so on Linux use the window or the tray instead.

## 3. Get a voice

MiraSpeak's built-in engine, **Kokoro**, is free and runs entirely on your PC.

Before the models are downloaded, Speak shows that Kokoro isn't ready, with **Check again**:

![First launch, Kokoro not downloaded](images/quick-start/04-first-launch.png)

1. In the MiraSpeak window, open **Voice engines → In-app** and set **Engine** to **Kokoro**.
2. Select **Download models**. This is a one-time download.

   ![Kokoro page with Download models](images/quick-start/05-kokoro.png)

The default voice (Heart) works as soon as the download finishes. To pick from more voices later, select **Refresh voices** and check the ones you want to see on Speak. Kokoro Japanese voices also need **Download Japanese dictionaries** on the same page. Other languages don't use it. More engine setup is in [Voice engines](../architecture/engines.md).

## 4. Speak your first line

1. Open **Speak**.
2. Type a line and press **Enter**.

   ![A line on Speak](images/quick-start/06-first-line.png)

You hear it on your computer's default speakers or headphones. To use a different device, open **Output → Audio devices** and change **Output**.

![Audio devices](images/quick-start/07-audio-devices.png)

The voice bar under the line box starts with the **Persona** and **Preset** chips. To change the voice, pick **Manual** on the **Preset** chip, then use the chips that follow: **Engine**, **Character**, **Voice**, **Speech**, **Style**, **Pace**, and **Pitch**. Which chips appear depends on the engine and voice. **Translate** sits after Pitch.

## 5. Speak while gaming

Press **Ctrl+Shift+Space** to open the quick bar over your game.

With **Stay open** off (the default):

| Key | What it does |
| --- | --- |
| **Enter** | Speak and close the bar |
| **Shift+Enter** | Speak and keep the bar open |
| **Esc** | Close the bar |
| **/** | Search commands (switch voice, preset, theme, and more) |
| **Ctrl+/** | Pick an engine (in Manual) or a preset |

With **Stay open** on, Enter keeps the bar up after speech. Esc and the hotkey still close it.

Focus goes back to your game when the bar closes. You can change the shortcut and Stay open under **Interface → Quick bar**. The full slash-command catalog is in [CLI, commands, and the window](cli.md).

## 6. Save voices as presets

Open **Voice → Presets** and fill a slot with an engine, voice, style, pace, and pitch. **Play** previews it.

Press **Ctrl+Shift+1** through **Ctrl+Shift+9** to switch to slots 1–9 from anywhere, even in game. **Add preset** adds more slots, which you pick from the **Preset** chip or with `/preset`.

## 7. Next steps

- To send the voice to OBS on Windows, use **Set up VB-Cable** as described in [Audio and captions](../architecture/outputs.md). Set **Monitor** to your headphones to keep hearing yourself (Monitor is a second copy of each line, played alongside Output).
- To send the voice to OBS on Linux, open **Output → Audio devices** and select **Create virtual output**. It creates a `MiraSpeak` sink and switches Output to it. In OBS, add an **Audio Input Capture** and pick **Monitor of MiraSpeak**.
- For on-stream captions, turn on the caption file as described in [Audio and captions](../architecture/outputs.md). Point an OBS Text source at the file.
- For Azure voices and translation, see [Azure Speech + Translator](azure-speech-translator.md).
- For Japanese character voices, see [Voice engines](../architecture/engines.md#voicevox).
- To bridge Twitch chat or send mouth shapes to a VMC receiver, see [Integrations](../architecture/integrations.md).
- To let an AI agent drive MiraSpeak, see [Connect an MCP client](mcp-clients.md). To reach it from a cloud agent, see [Remote MCP via Cloudflare](remote-mcp-cloudflare.md). To run with no window, see [CLI mode](cli.md).
- To change the theme, use **Interface → Appearance** or `/theme`. It switches between the built-in themes, most of them in Light, Dark, and Midtones. To make your own, select **Duplicate** and edit the copy.
- To add a module, see [Integrations](../architecture/integrations.md#modules). Only install modules from people you trust.

## Tips for typing lines

- `[date]`, `[time]`, and stray `[brackets]` are covered in [Marker tokens](markers.md).
- Turn on **Split sentences** under the line box to speak each sentence as its own line.
- Start a line with `//` to speak a leading slash instead of running a command.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| You hear nothing | Check **Output → Audio devices → Output**. Monitor is only an extra copy of Output, so it never plays on its own. Select **Rescan** after plugging in a device or changing the system default. |
| A line fails with "Output device '…' not found" | A device you picked is unplugged or gone. Pick another, or **System default**. |
| A line fails with "No default output device" | Windows has no default playback device. Set one in Windows sound settings, then **Rescan**, or pick a device under **Output**. |
| The Speak button is off | The engine isn't ready yet. Download the Kokoro models (step 3), or start VOICEVOX if you're using it. |
| The quick bar shortcut does nothing | Another app already uses it. Pick a different one under **Interface → Quick bar → Hotkey**. |
| The window doesn't appear when you start MiraSpeak | It's already running in the tray. Select the tray icon. |
