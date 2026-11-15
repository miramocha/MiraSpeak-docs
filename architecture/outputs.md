# Outputs: Audio devices and text output

Both are pages under Output in settings.

## Audio devices

Audio devices picks the **Output** device and an optional **Monitor** copy. On Windows, **Set up VB-Cable** installs a virtual cable for OBS. On Linux, **Create virtual output** makes a PulseAudio or PipeWire sink and selects it as Output. The `audio/` layer is described in [Layers](layers.md#audio-and-mouth). Setup steps are in the [quick start](../guides/quick-start.md).

## Text output

When a line starts playing, MiraSpeak writes it to the caption file with markers stripped, for stream overlays. With **Split sentences** on, that's one sentence at a time. The write is atomic. The file clears 2 seconds after MiraSpeak is idle (the last line has ended and the queue is empty). **Caption file** turns the write on. For a [translated](translation.md) line, **Caption text** is **Translated** (the default), **Original**, or **Both** (the original, then the translation on the next line).
