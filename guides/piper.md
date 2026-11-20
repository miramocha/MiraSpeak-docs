# Piper voices

Piper is a local voice engine: English voices that run on your CPU, one downloaded file per speaker. It used to be built into MiraSpeak. It's now the official **Piper** module, which adds the engine back and keeps its settings page where it was.

- It's free and offline once a voice is downloaded. Nothing you say leaves your computer.
- It needs MiraSpeak with engine modules (module API 1 with the `voice_engine` permission). An older MiraSpeak refuses the install with "Unknown permission: voice_engine."

## 1. Install the module

1. Open **Integrations → Modules → Install from URL…** and paste:
   https://github.com/miramocha/MiraSpeak-modules/tree/v0.3.0/piper
2. The consent screen lists what the module asks for: **voice engine** and **network**. It also lists the Python packages it installs into its own folder, `piper-tts` and `pathvalidate`. Select **Install and enable**. MiraSpeak downloads the packages from PyPI (about 33 MB).

Piper uses the onnxruntime and numpy that MiraSpeak already has for Kokoro, so it doesn't download its own copies.

## 2. Download a voice

1. Open **Voice engines → In-app** and pick **Piper** in the **Engine** list. It sits with Kokoro and Windows, in name order.
2. Until a voice is installed, the list shows **Piper · Piper voices not downloaded** with a warning dot.
3. Pick a voice under **Voice** and select **Download**. **Refresh list** downloads the latest voice list, and **Cancel** stops a download.
4. The voice appears under **Installed**, and Piper is ready to speak. Pick it in the Speak engine chip, a preset, or a persona like any other engine.

Under the voice list, MiraSpeak shows the same **Speech cache** and **Generate ahead** settings as its other engines.

To delete a voice, select **Remove** on its row. Piper stays ready while at least one installed voice is on disk.

## Coming from the built-in Piper

- The engine id is still `piper`, so personas, presets, and the live voice keep pointing at Piper.
- Downloaded voices stay in the same folder (`%LOCALAPPDATA%\MiraSpeak\piper`), so you don't need to download them again.
- Lines Piper already cached still play from the cache.
- Your speech cache and generate-ahead settings carry over.

Until the module is installed and enabled, a line that uses Piper speaks with Kokoro's first voice instead. The line's row says "piper is not installed or enabled; using Kokoro". Your settings don't change, so the line uses Piper again once the module runs.

The old Piper runtime download in `%LOCALAPPDATA%\MiraSpeak\support\piper` isn't used anymore. You can delete that folder.

## Troubleshooting

| Problem | What to do |
| --- | --- |
| Piper isn't in the In-app Engine list | The module isn't running. Enable it on **Integrations → Modules**. |
| "Piper packages not installed. Reinstall the Piper module." | Remove the Piper module and install it again. |
| A voice row says **missing files** | Pick it under **Voice** and select **Download** again. |
| "Removed … The voice files are still on disk." | Windows still had the voice open. Restart MiraSpeak, then delete it from the voices folder. |
| "This Piper voice needs the … package." | That voice needs an extra phonemizer package the module doesn't install. Pick another voice. |
| "Piper currently supports English voices only." | The voice isn't English, for example one added to settings by hand. Pick one from the **Voice** list, which offers only English voices. |

## License

Piper (piper-tts) is GPL-3.0-or-later and installs from PyPI. Each voice keeps the license published with its files. The module code is MIT.
