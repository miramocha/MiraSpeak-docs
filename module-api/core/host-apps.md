# For host apps (may change)

This page is for an app that loads modules, not for module authors. These names can change in any CetiModule release. Code: `cetimodule.host`.

Module authors import from `cetimodule`, or from their app's API module such as `miraspeak.modules.api`. A module that imports `cetimodule.host` isn't using a supported API.

## What a host app passes in

- **Permissions and flags.** A `ManifestSchema` with every permission the app grants (`SHARED_PERMISSIONS` plus its own) and each extra `true` / `false` manifest key with its default.
- **Imports.** A `ModuleImporter` with the package namespace module code loads under and the app's `Module` base class.
- **Folders and secrets.** The settings folder and the environment prefix for secrets.
- **Downloads.** The User-Agent and an `ArchiveDownload` function for GitHub installs, so the app keeps one HTTP client and one size limit.

## Main names

| Name | Role |
| --- | --- |
| `ModuleRuntime` | One module's thread: import, callbacks, timers, and the error limit |
| `RuntimeContext` | The `ModuleContext` for one runtime. An app subclasses it to add methods. |
| `ModuleHost`, `ModuleState`, `PageUpdate`, `PageUpdateKind` | What a runtime calls on the app, and the page updates it posts |
| `load_module_manifest`, `parse_module_manifest`, `ModuleManifestDoc` | Manifest parsing |
| `discover_modules`, `ModuleSource`, `ModuleLoadError` | Finding and loading module folders |
| `StagedModule`, `ModuleInstallError`, `GithubSource` | .zip and GitHub installs |
| `install_requires`, `current_target`, `PackageTarget` | Package installs from pylock files |
| `ModuleSecrets`, `SecretStore` | Per-module secrets in the OS credential store |

The full list is `cetimodule.host.__all__`.
