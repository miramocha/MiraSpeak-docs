# Install a module from GitHub

MiraSpeak installs a module from a public GitHub repository, either the whole repository or one folder in it. One repository can hold many modules, one per folder, as [MiraSpeak-modules](https://github.com/miramocha/MiraSpeak-modules) does. The in-app install steps are in [Integrations](../architecture/integrations.md#modules). Official module URLs are in [Modules](../architecture/modules.md#official-modules). This page is for publishing a module and for the rules the install link follows.

## Links

On **Integrations → Modules**, select **Install from URL…** and paste one of these:

| Link | Installs |
| --- | --- |
| `https://github.com/<owner>/<repo>` | The repository root, from the default branch |
| `https://github.com/<owner>/<repo>/tree/<ref>/<folder>` | `<folder>` at the tag, branch, or commit `<ref>` |

For example, `https://github.com/miramocha/MiraSpeak-modules/tree/v0.1.0/timer` installs the `timer` folder at tag `v0.1.0`.

- `module.toml` sits at the top of the folder. Only that folder installs.
- For a branch such as `feat/hello`, the link can't show where the ref ends and the folder starts. Check **Tag or commit** and **Folder** in the Install from GitHub dialog before you install. The dialog fields are **Repository**, **Tag or commit**, and **Folder**, and the accept button is **Download**. Leave **Tag or commit** empty for the default branch, and leave **Folder** empty when `module.toml` is at the repository root.
- A private repository reads as a missing ref. Only public repositories install.
- A folder with `..`, a backslash, or a colon is refused.

## What happens

MiraSpeak resolves the ref to one commit, downloads that commit's archive from GitHub, and runs it through the same checks as a .zip install. There's no git clone, so no git hooks or filters run. The checks, consent, and swap-in are in [Modules](../architecture/modules.md#install-and-consent).

The Modules row records the repository URL, the commit, the ref as typed, and the folder. **Update** asks for the ref again; MiraSpeak doesn't check for updates in the background.

## Publish a module

1. Put each module in its own folder with its `module.toml`.
2. Tag a release, such as `v0.1.0`.
3. Share the folder link at that tag: `https://github.com/<owner>/<repo>/tree/v0.1.0/<folder>`.

A tag link installs the same files every time. A branch link installs whatever the branch holds when the user installs or updates.
