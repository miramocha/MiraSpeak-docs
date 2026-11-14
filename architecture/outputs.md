# Outputs: audio devices and text output

Both are pages under Output in settings.

## Audio devices

Audio devices picks the **Output** device and an optional **Monitor** copy, and can create a virtual output for OBS. The `audio/` layer is described in [Layers](layers.md#audio-and-mouth). Setup steps are in the [quick start](../guides/quick-start.md).

## Text output

Writes the currently playing line to a text file for stream overlays. The write is atomic, and the file clears a couple of seconds after nothing is playing. When a line was [translated](translation.md), the caption can be the translated text (default), the original, or both.
