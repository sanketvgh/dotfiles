# dotfiles

Managed with [chezmoi](https://www.chezmoi.io). Currently holds the VS Code **Zen** profile only, for Windows and Linux (zsh).

## What is managed

| Path | Purpose |
| --- | --- |
| `.chezmoitemplates/vscode/zen/settings.json` | Zen settings (shared across OSes) |
| `.chezmoitemplates/vscode/zen/terminal-profiles` | Claude/Codex/Shell terminal profiles: `pwsh` on Windows, `zsh` on Linux |
| `.chezmoitemplates/vscode/oxfmt-path` | Per-OS path to the global `oxfmt` binary |
| `.chezmoitemplates/vscode/zen/keybindings.json` | Zen keybindings |
| `.chezmoitemplates/vscode/zen/extensions.txt` | Zen extensions |
| `.chezmoiscripts/*-vscode-zen.*` | Writes settings and keybindings into the Zen profile folder |
| `.chezmoiscripts/*-vscode-extensions.*` | Installs missing Zen extensions |

The Zen profile folder has a random ID per machine (`Code/User/profiles/<id>`), so the scripts look it up in VS Code's `globalStorage/storage.json` instead of using a fixed chezmoi target. Each script reruns only when the rendered content changes.

## Setup on a new machine

1. Install VS Code, start it once, and create a profile named **Zen**.
2. Install chezmoi and point it at this folder in `~/.config/chezmoi/chezmoi.toml`:

   ```toml
   sourceDir = "~/Projects/dotfiles"

   # Windows only: run .ps1 scripts with PowerShell 7 so UTF-8 names render correctly
   [interpreters.ps1]
   command = "pwsh"
   args = ["-NoLogo", "-NoProfile"]
   ```

3. Preview, then apply:

   ```sh
   chezmoi diff
   chezmoi apply
   ```

Linux needs `python3` (to read `storage.json`) and the `code` CLI on `PATH`.

## Not managed here

The Zen profile expects these to exist on the machine:

- Fonts (install the variable versions, per the projects' READMEs):
  - [Cascadia Code](https://github.com/microsoft/cascadia-code) ([v2407.24](https://github.com/microsoft/cascadia-code/releases/tag/v2407.24)): editor (`Cascadia Code`) and terminal (`Cascadia Code NF`)
  - [Inter](https://github.com/rsms/inter) ([v4.1](https://github.com/rsms/inter/releases/tag/v4.1)): Markdown preview; on Windows use the hinted static `Inter.ttc`
- `oxfmt` installed globally with pnpm
- `~/.markdownlint.json`

## Workflow

VS Code writes to the live `settings.json` (for example when toggling the status bar). Those edits are not copied back automatically. Edit the templates here, then run `chezmoi apply`. Use `chezmoi diff` first to spot drift.
