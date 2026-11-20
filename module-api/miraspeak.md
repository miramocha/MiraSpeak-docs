# MiraSpeak module API

`miraspeak.modules.api` is the only MiraSpeak import module code uses. It re-exports the [author API](core/author-api.md) and `ui` from CetiModule as the same objects, and adds speech: permissions, context methods, callbacks, and the types they carry.

```python
from miraspeak.modules.api import LIVE, Module, ModuleContext, ModuleError, ui
```

`API_VERSION` is 1. A manifest's `api` must match it; a module built for another version shows *Needs update* and doesn't load. Additive changes keep the version.

## Permissions

`PERMISSIONS` is the set a MiraSpeak manifest may ask for: the 3 [shared permissions](core/manifest.md#permissions) plus `speak`, `observe_lines`, `observe_chat`, `stop_all`, `audio`, `observe_playback`, `style_lines`, `filler`, and `voice_engine`. What each one allows is in [Modules](../architecture/modules.md#permissions). The tables below name the permission each method and callback needs.

## Manifest flag

MiraSpeak reads one extra manifest key, `post_to_chat` (`true` / `false`, default `true`). It sets the default of the module's **Post to chat** setting, which the user can change. When the setting is on, the module's spoken lines are posted through TTS → Chat.

## Context

In MiraSpeak, `on_start` receives a `ModuleContext` with the shared members plus these.

| Method | Needs | Returns |
| --- | --- | --- |
| `speak(text, *, voice=LIVE, split=None)` | `speak` | `SpeakResult`. Queues `text` like a typed line. `split=None` follows the Split Sentences setting. |
| `speak_lines(pieces, *, voice=LIVE)` | `speak` | `SpeakResult`. Queues exactly `pieces` as one stack; blank pieces are skipped. |
| `stop(result)` | | Stops the lines of one `SpeakResult` |
| `stop_mine()` | | Stops every line this module queued |
| `stop_all()` | `stop_all` | Stops every line, not only this module's |
| `voices()` | | `tuple[VoiceOption, ...]`: the live voice and the presets |
| `playback()` | | `Playback` for the line playing now, or `None` |
| `queue_depth()` | | How many lines are waiting |
| `cached(texts, *, voice=LIVE)` | | `tuple[bool, ...]`: which texts already have a speech-cache file for `voice`. File checks only. With translation on, a text counts only when its translation is cached too. |
| `set_mouth_profile(profile)` | `observe_playback` | Starts `on_mouth` with that `MouthProfile`; `None` stops it. Doesn't wait for MiraSpeak, so it's safe in `on_start`. A missing permission is logged, not raised. |
| `list_personas()` | `filler` | `tuple[PersonaOption, ...]` |
| `list_filler_cues(persona_id="")` | `filler` | `tuple[FillerCueInfo, ...]`, or a `FillerResult` refusal. An empty id uses the live voice. |
| `start_filler(*, persona_id="", cue="", timeout_s=None)` | `filler` | `FillerResult`. Starts a cache-only filler session; an empty cue plays the set's default. |
| `cue_filler(*, persona_id="", cue="", session_id="", timeout_s=None)` | `filler` | `FillerResult`. Switches the session to `cue` at the next take and restarts its timeout. |
| `stop_filler(session_id="")` | `filler` | `False` when that session wasn't playing |
| `engine_config(engine_id)` | `voice_engine` | A copy of the settings table of an engine this module added |
| `set_engine_config(engine_id, *, voices)` | `voice_engine` | Replaces that engine's voice list (Speak alias → the engine's own voice id) |

`speak` and `speak_lines` raise `SpeakRefusedError` when the line isn't queued: empty text, an empty preset, a full queue, or calls that come too fast. `open_url` shares the speak rate limit, so it can raise `SpeakRefusedError` too.

### Voices

`VoiceRef` names a configured voice. `LIVE` is the live voice (the current chips). `VoiceRef.preset(n)` is preset slot *n*, 1 or higher. `VoiceRef.parse("preset:3")` reads the `id` form back; anything else is `LIVE`. A module speaks with the voices the user set up and never changes them.

## Callbacks

| Callback | Needs | Called when |
| --- | --- | --- |
| `on_line(event: LineEvent)` | `observe_lines` for other sources' lines | A line changes state |
| `on_style(request: StyleRequest) -> str \| None` | `style_lines` | A live line is about to be spoken. Return `request.spoken` unchanged, that text with one leading `[id]` from `request.styles`, or `None` to leave the choice to others. MiraSpeak waits up to 1 second. |
| `on_chat_message(event: ChatMessageEvent)` | `observe_chat` | A live-chat message passes Share |
| `on_audio(stream: AudioStream)` | `audio` | A line starts playing. Hand `stream` to a thread that reads it, then return. |
| `on_playback_signal(signal: PlaybackSignal)` | `observe_playback`, plus `observe_lines` for other sources' lines | A 60 fps frame of the playing line. A slow module skips to the newest frame. |
| `on_mouth(line: MouthLine)` | `observe_playback` | A line starts playing, after `set_mouth_profile`. Hand `line` to a thread, then return. |
| `on_voices_changed()` | | The voices a module can speak with changed: the live voice, a preset, the engine or model, style, pace, speech language, the engine list, or the translation lookup |
| `engines() -> Sequence[type[Adapter]]` | `voice_engine` | Once, after `on_start`. Returns the voice engine adapters the module adds. |

Engine modules are covered in [Modules](../architecture/modules.md#engine-modules) and [Voice engines](../architecture/engines.md#engine-modules). The [Piper module](https://github.com/miramocha/MiraSpeak-modules/tree/v0.3.0/piper) is a working example; its setup is in the [Piper guide](../guides/piper.md).

## Types

### Lines

`LineEvent` is one step of a line's lifecycle.

| Field | Meaning |
| --- | --- |
| `line_id` | The line |
| `state` | A `LineState` |
| `text` | The line's text |
| `mine` | `True` for lines this module queued |
| `voice_label` | The voice's display label |
| `batch_id`, `batch_index`, `batch_size` | The line's place in a stack, when it has one |
| `duration_s` | Clip length in seconds, or `None` |
| `error` | Error text, or `None` |
| `silent` | `True` for a line rendered to a file and never played. `DONE` comes without `PLAYING`. |
| `persona_id`, `persona_name`, `persona_color` | The persona that spoke it, when there is one |

A module that reacts to "a line finished" checks `silent`.

`SpeakResult` holds `line_ids` (in order) and `batch_id`, which is set when one call queued 2 or more lines.

### Chat

`ChatMessageEvent` has `platform`, `channel`, `author_login`, `author_id`, `display_name`, `badges` (`ChatBadges`), `text`, `timestamp`, `is_self`, and `message_id`.

### Playback and mouth

| Type | Fields |
| --- | --- |
| `Playback` | `line_id`, `position_s` (seconds heard), `duration_s`, `mine` |
| `PlaybackSignal` | `line_id`, `position_s`, `duration_s`, `level` (clip loudness 0–1), `mouth` (weights for `a i u e o`, 0–1), `mine` |
| `MouthProfile` | `mode` (`"arkit"` or `"aeiou"`), `source` (`"audio"`, `"text"`, or `"hybrid"`), `names` |
| `MouthLine` | `line_id`, `mine`, `track` (`MouthTrack`), `position()`, `over` (a `threading.Event`) |

`"arkit"` gives the 8 Perfect Sync keys (`JawOpen` and the rest) and ignores `names`. `"aeiou"` gives one weight per vowel named in `names`, such as `{"A": "Mouth_A"}`. Unknown values fall back to `"arkit"` and `"audio"`.

A `MouthTrack` has `keys`, `fps`, `duration`, `at(seconds)` (weights, all zero outside the clip), and `level_at(seconds)`. Read `line.track.at(line.position())` from a thread the module owns, about every 16 ms, until `line.over` is set. `position()` returns `None` while nothing is audible.

### Audio

An `AudioStream` has `line_id`, `mine`, `read_frame()`, and `close()`. `read_frame()` returns the next 20 ms as `AUDIO_FRAME_BYTES` (3,840) bytes of signed 16-bit little-endian stereo at `AUDIO_SAMPLE_RATE` (48,000 Hz), silence while the take is still arriving, and `b""` once the line ends or stops. Read from one thread. A slow reader never delays playback.

### Filler

| Type | Fields |
| --- | --- |
| `PersonaOption` | `id`, `name`, `vacant` |
| `FillerCueInfo` | `name`, `style`, `phrases`, `takes` (cached), `default` |
| `FillerResult` | `ok`, `session_id` (set on a start), `code` (empty when accepted), `message` |

## Helpers

| Name | Purpose |
| --- | --- |
| `split_sentences` | MiraSpeak's sentence splitter |
| `has_unclosed_marker` | `True` when text has an unfinished `[` marker |
| `webhook_url_allowed` | `True` for an `https` URL with a host, or `http` to localhost, 127.0.0.1, or ::1 |
| `ReadAlong` | Tracks which of a module's queued lines are still in a read, and whether it may queue another sentence |
| `Follow` | What to show after one of those owned line events: highlight, show, fill, and paused |
| `LineState`, `ChatBadges` | Types used by `LineEvent` and `ChatMessageEvent` |
