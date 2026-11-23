# Outputs: Audio devices and text output

Both are pages under Output in settings.

## Audio devices

Audio devices picks the **Output** device and an optional **Monitor** copy. Monitor is a second
copy of each line, played alongside Output; it never plays on its own.

Output starts on **System default** (your default speakers). On Windows, **Output → Audio
devices → Set up VB-Cable** installs [VB-Audio Virtual Cable](https://vb-audio.com/Cable/) and
switches Output to `CABLE Input` after a Rescan. OBS and Warudo use `CABLE Output` as their
microphone.

On Linux, **Create virtual output** makes a PulseAudio or PipeWire sink and selects it as
Output. In OBS, add an **Audio Input Capture** and pick **Monitor of MiraSpeak**. The `audio/`
layer is described in [Layers](layers.md#audio-and-mouth).

## Text output

On **Text output**, enable the caption file (**Caption file**) to rewrite it when each line
starts playing, with markers stripped, for stream overlays. With **Split sentences** on, that's
one sentence at a time. The write is atomic. The file clears 2 seconds after MiraSpeak is idle
(the last line has ended and the queue is empty), so OBS can show the last line. Point an OBS
Text (GDI+) source at it. The default path is `%LOCALAPPDATA%\MiraSpeak\latest.txt`.

For a [translated](translation.md) line, **Caption text** is **Translated** (the default),
**Original**, or **Both** (the original, then the translation on the next line).
