# Build pylock files

A module that needs Python packages lists them in `requires` and ships one `pylock.<target>.toml` per platform. MiraSpeak installs the wheels from those locks, with no pip or Python of its own. The file rules are in [Module manifest](../module-api/core/manifest.md#packages).

These steps use [uv](https://docs.astral.sh/uv/).

## 1. Pin the packages

List each package once, as an exact pin, in `requirements.in`:

```text
piper-tts==1.8.0
pathvalidate==3.3.1
```

Put the same pins in `module.toml`:

```toml
requires = ["piper-tts==1.8.0", "pathvalidate==3.3.1"]
```

## 2. Lock each target

Run `uv pip compile` once per target, with wheels only and the pylock format:

```bash
uv pip compile requirements.in \
  --python-version 3.12 \
  --python-platform <platform> \
  --only-binary :all: \
  --format pylock.toml \
  -o pylock.cp312-<target>.toml
```

| File | `<platform>` |
| --- | --- |
| `pylock.cp312-win_amd64.toml` | `x86_64-pc-windows-msvc` |
| `pylock.cp312-macos_arm64.toml` | `aarch64-apple-darwin` |
| `pylock.cp312-linux_x86_64.toml` | `x86_64-manylinux_2_28` |

These are the `--python-platform` values the official modules use. The Linux lock targets `manylinux_2_28`. A machine with no lock for its target can't install the module.

## 3. Check the locks

Compare each lock with the rules in [Module manifest](../module-api/core/manifest.md#lock-files). The install refuses a lock that breaks one, such as a lock that pins a different version than `requires`.

## Packages MiraSpeak already ships

A module's packages load after MiraSpeak's own, so a package MiraSpeak already has wins. A lock can leave such packages out. The Piper module leaves out `onnxruntime` and `numpy`, which `piper-tts` needs and MiraSpeak already ships, by compiling with `--no-deps`:

```bash
uv pip compile requirements.in --no-deps \
  --python-version 3.12 \
  --python-platform x86_64-pc-windows-msvc \
  --only-binary :all: \
  --format pylock.toml \
  --no-header \
  -o pylock.cp312-win_amd64.toml
```

The official Piper macOS lock also sets `MACOSX_DEPLOYMENT_TARGET=14.0` so the wheels match MiraSpeak's onnxruntime.

## 4. Ship them

Commit `requirements.in` and the lock files next to `module.toml`. The consent screen lists the `requires` packages, and an update that changes them asks again.
