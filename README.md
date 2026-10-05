# dotfiles

Managed with [chezmoi](https://www.chezmoi.io). Holds the VS Code **Zen** profile, for Windows and Linux (zsh).

## What is managed

| Path | Purpose |
| --- | --- |
| `.chezmoitemplates/vscode/zen/settings.json` | Zen settings (shared across OSes) |
| `.chezmoitemplates/vscode/zen/terminal-profiles` | Claude/Codex/Shell terminal profiles: `pwsh` on Windows, `zsh` on Linux |
| `.chezmoitemplates/vscode/oxfmt-path` | Per-OS path to the global `oxfmt` binary |
| `.chezmoitemplates/vscode/zen/keybindings.json` | Zen keybindings |
| `.chezmoitemplates/vscode/zen/extensions.txt` | Zen extensions |
| `.chezmoidata.toml` | Name of the VS Code profile to apply to (`vscodeProfile`) |
| `.chezmoiscripts/*-vscode-zen.*` | Writes settings and keybindings into the Zen profile folder |
| `.chezmoiscripts/*-vscode-extensions.*` | Installs missing Zen extensions |

The Zen profile folder has a random ID per machine (`Code/User/profiles/<id>`), so the scripts look it up in VS Code's `globalStorage/storage.json` instead of using a fixed chezmoi target. Each script reruns only when the rendered content changes.

## Setup on a new machine

Both systems follow the same steps: install the tools, clone this repo to `~/Projects/dotfiles`, point chezmoi at it, create the **Zen** profile, then apply.

### Windows (PowerShell 7)

Requires [PowerShell 7](https://github.com/PowerShell/PowerShell) (`pwsh`), VS Code with the `code` command on `PATH`, and git.

```powershell
# 1. Install chezmoi
scoop install chezmoi          # or: winget install twpayne.chezmoi

# 2. Clone the repo
git clone <repo-url> "$HOME\Projects\dotfiles"

# 3. Point chezmoi at the repo and run .ps1 scripts with PowerShell 7
New-Item -ItemType Directory -Force "$HOME\.config\chezmoi" | Out-Null
@'
sourceDir = "~/Projects/dotfiles"

[interpreters.ps1]
command = "pwsh"
args = ["-NoLogo", "-NoProfile"]
'@ | Set-Content "$HOME\.config\chezmoi\chezmoi.toml"

# 4. Create the Zen profile (opens a VS Code window), then close that window
code --profile Zen --new-window

# 5. Preview, then apply
chezmoi diff
chezmoi apply
```

The `[interpreters.ps1]` block matters: without it chezmoi runs scripts with Windows PowerShell 5.1, which garbles UTF-8 names such as `Claude · Continue`.

### Linux (zsh)

Requires zsh, `python3` (used to read VS Code's `storage.json`), VS Code with the `code` command on `PATH`, and git.

```sh
# 1. Install chezmoi (or use your distro's package)
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b "$HOME/.local/bin"

# 2. Clone the repo
git clone <repo-url> ~/Projects/dotfiles

# 3. Point chezmoi at the repo
mkdir -p ~/.config/chezmoi
printf 'sourceDir = "~/Projects/dotfiles"\n' > ~/.config/chezmoi/chezmoi.toml

# 4. Create the Zen profile (opens a VS Code window), then close that window
code --profile Zen --new-window

# 5. Preview, then apply
chezmoi diff
chezmoi apply
```

### What `chezmoi apply` does

1. Finds the Zen profile folder through `storage.json` and writes `settings.json` and `keybindings.json` into it.
2. Installs any Zen extensions that are missing.

If VS Code has never been started, or the Zen profile does not exist yet, the script stops with a message explaining what to do. Run `chezmoi apply` again after fixing it.

### Keeping machines in sync

After changing the templates on one machine and pushing:

```sh
git -C ~/Projects/dotfiles pull
chezmoi diff
chezmoi apply
```

Scripts rerun only when the content they write has changed, so `chezmoi apply` is safe to run any time.

## My tools

Installed by hand, not by chezmoi.

| Tool | How I install it | Notes |
| --- | --- | --- |
| CLI apps (Windows) | [scoop](https://scoop.sh) | For example `scoop install chezmoi` |
| [pnpm](https://pnpm.io) | `corepack install -g pnpm@latest`, then `pnpm setup` once | Always the latest pnpm through corepack; never `npm i -g` |
| Global JS tools | `pnpm add -g <tool>` | For example `pnpm add -g oxfmt`, which the Zen formatter uses |

`pnpm setup` adds `PNPM_HOME` to the shell config (`~/.zshrc` on Linux, the user environment on Windows). Open a new terminal afterwards. Corepack is not bundled with every Node.js release; install it separately if the `corepack` command is missing.

## Not managed here

The Zen profile expects these to exist on the machine:

- Fonts (install the variable versions, per the projects' READMEs):
  - [Cascadia Code](https://github.com/microsoft/cascadia-code) ([v2407.24](https://github.com/microsoft/cascadia-code/releases/tag/v2407.24)): editor (`Cascadia Code`) and terminal (`Cascadia Code NF`)
  - [Inter](https://github.com/rsms/inter) ([v4.1](https://github.com/rsms/inter/releases/tag/v4.1)): Markdown preview; on Windows use the hinted static `Inter.ttc`
- `oxfmt`, installed globally with pnpm (see [My tools](#my-tools))

## Workflow

VS Code writes to the live `settings.json` (for example when toggling the status bar). Those edits are not copied back automatically. Edit the templates here, then run `chezmoi apply`. Use `chezmoi diff` first to spot drift.
