# Module manifest

Every module has a `module.toml` at the top of its folder. CetiModule reads it before anything in the module runs. The host app decides which permissions and extra flags a manifest may use. Code: `cetimodule.manifest`.

```toml
id = "hello"
name = "Hello"
version = "0.1.0"
api = 1
entry = "main:HelloModule"
description = "Say a greeting with the live voice."
author = "<your-name>"
permissions = ["speak"]
```

## Fields

| Key | Required | Type | Rule |
| --- | --- | --- | --- |
| `id` | Yes | Text | 2–32 characters: lowercase letters, digits, or hyphens, starting with a letter |
| `name` | Yes | Text | Display name |
| `version` | Yes | Text | The module's own version, such as `0.1.0` |
| `api` | Yes | Integer | The host app's module API version, 1 or higher. The host compares it to its own version. |
| `entry` | Yes | Text | `file:Class` inside the module folder, such as `main:HelloModule`. The file part may be dotted (`pkg.main:HelloModule`). |
| `description` | No | Text | Default `""` |
| `author` | No | Text | Default `""` |
| `permissions` | No | List of names | Default `[]`. Every name must be one the host app grants. |
| `requires` | No | List of exact pins | Default `[]`. Python packages the module installs. See [Packages](#packages). |
| `keep_page` | No | `true` / `false` | Default `false`. The page stays while the module is stopped, with a switch that turns it on and off. |
| `switch_label` | No | Text | The sentence next to that switch. Empty shows "On". |
| `default_enabled` | No | `true` / `false` | Default `false`. Bundled modules only: the module starts enabled with everything it asks for granted, until the user turns it off. An installed module's value is ignored. |

Text values are trimmed. With `keep_page`, the host calls `page()` before the module starts, so `page()` must not start work.

A host app can also declare extra `true` / `false` keys. MiraSpeak declares `post_to_chat`; see the [MiraSpeak module API](../miraspeak.md#manifest-flag).

## Permissions

`permissions` lists what the module asks for. The user sees the list at consent and grants it before the module starts.

CetiModule defines 3 permissions that mean the same in every app:

| Permission | Constant | Meaning |
| --- | --- | --- |
| `network` | `PERMISSION_NETWORK` | Consent for network access. `ctx.open_url` also checks it. |
| `files` | `PERMISSION_FILES` | Consent for file access. The module's own data folder (`ctx.data`) doesn't need it. |
| `mcp_tools` | `PERMISSION_MCP_TOOLS` | Adds the module's tools and instructions to the app's MCP server ([MCP tools](mcp-tools.md)) |

`SHARED_PERMISSIONS` holds all 3. A host app adds its own; MiraSpeak's are listed in [Modules](../../architecture/modules.md#permissions). A Python module runs with the same access as the app, so `network` and `files` are consent only. A network grant isn't limited to one host.

## Packages

`requires` lists Python packages as exact pins: `name==version`, or `name[extra,...]==version`.

```toml
requires = ["piper-tts==1.8.0", "pathvalidate==3.3.1"]
```

- **Exact pins only.** A range such as `>=1.0` fails the manifest.
- **One entry per package.** Names are compared after normalization (lowercase, with runs of `-`, `_`, and `.` folded to `-`), so `Foo_Bar` and `foo-bar` are the same package.

### Lock files

A module with `requires` ships one lock file per target it supports, named `pylock.<target>.toml`, such as `pylock.cp312-win_amd64.toml`. Each lock holds the full set of packages for that one Python version and platform. To make them, see [Build pylock files](../../guides/module-packages.md).

The target is `cp3<minor>-<platform>`. The host picks the lock for the machine it runs on:

| Machine | Target (Python 3.12) |
| --- | --- |
| Windows x64 | `cp312-win_amd64` |
| Windows Arm64 | `cp312-win_arm64` |
| Windows 32-bit | `cp312-win32` |
| macOS Apple silicon | `cp312-macos_arm64` |
| macOS Intel | `cp312-macos_x86_64` |
| Linux x64 (glibc) | `cp312-linux_x86_64` |
| Linux Arm64 (glibc) | `cp312-linux_aarch64` |

On Linux, only glibc systems match `manylinux` wheels. A musl system finds no matching platform wheel.

MiraSpeak runs CPython 3.12: the project's `.python-version` is 3.12, and uv installs that version. Official modules ship `cp312` locks only. The lock target follows the running interpreter, so a newer local Python looks for a different lock name.

The install checks each lock:

- **Lock and pins agree.** Every lock pins every `requires` entry at the same version.
- **Locks need pins.** A module with pylock files and no `requires` fails, and so does a module with `requires` and no lock.
- **Wheels only, from PyPI.** Every package has at least one `.whl`, its URL is `https://files.pythonhosted.org/...`, and it carries a `sha256` hash. `size`, when present, is a whole number.
- **One entry per package.** A package listed twice in one lock fails.

From each package's wheels, the installer takes the most specific one that loads on the machine: this CPython version, then `abi3`, then pure Python. Consent, download, and load order are in [Modules](../../architecture/modules.md#module-packages).

## Errors

A manifest that doesn't parse raises `ModuleManifestFormatError` with a message naming the key, such as `id 'Hello' must be 2–32 lowercase letters, digits, or hyphens, starting with a letter.` or `Unknown permission: speek.` The module doesn't install.
