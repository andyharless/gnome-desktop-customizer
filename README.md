# gnome-desktop-customizer

> **Note:** This skill was written with the help of Claude (Anthropic's AI assistant) and has not been fully reviewed or tested by a human. The commands and advice in it are believed to be correct but may contain errors, especially across different GNOME/Ubuntu versions. Review what Claude proposes before letting it make changes, and please open an issue if you find a mistake.

An [Agent Skill](https://www.anthropic.com/news/skills) for [Claude Code](https://claude.com/claude-code) (and any other harness that supports the same `SKILL.md` format) that teaches Claude how to customize the GNOME desktop and the Nautilus/Files file manager on Ubuntu — safely, and without you having to look up the right `gsettings` key or extension API every time.

Covers GNOME 42 (Ubuntu 22.04 LTS) and GNOME 46+ (Ubuntu 24.04 LTS and newer), and calls out the places those two diverge.

## What it's for

Ask Claude things like:

- "Add a 'New File' option to the right-click menu in Files"
- "Add a right-click action that opens the current folder in VS Code"
- "Show seconds in the clock" / any other `gsettings`-backed toggle
- "Set up a custom keyboard shortcut for X"
- "Install this GNOME Shell extension" / "list my enabled extensions"
- "Run this script automatically when I log in"

Instead of improvising or sending you off to search documentation yourself, Claude follows the skill's workflows: identify your GNOME version, make the change through the correct mechanism (Templates folder, Nautilus Scripts, `nautilus-python`, `gsettings`/`dconf`, `gnome-extensions`, autostart `.desktop` files), back up anything non-trivial to revert first, and verify the change actually took effect.

## Install

Clone directly into Claude Code's skills directory:

```bash
git clone https://github.com/andyharless/gnome-desktop-customizer.git ~/.claude/skills/gnome-desktop-customizer
```

For a project-local install instead of a global one, clone into `.claude/skills/gnome-desktop-customizer` inside your project.

Once installed, Claude will automatically use this skill for relevant requests — no further setup needed. You do not need to be running Claude Code *inside* GNOME for this to work; Claude just needs a shell on the Ubuntu machine you want to customize (for example, a remote or SSH session into that machine works too, though some changes need your graphical session's environment to apply).

## What it won't do without asking

- Any `sudo apt install` is called out and confirmed first (needed only for a few optional helper packages: `python3-nautilus`, `dconf-editor`, `gnome-tweaks`, `gnome-shell-extension-manager`).
- Nothing is edited outside `~/.config`, `~/.local/share`, or `~/Templates` (apart from backup files written to your home directory).
- Broader `dconf`/`gsettings` changes get backed up first so they can be reverted.

## Scope

This is about *your own* desktop and file-manager behavior — context menus, settings, shortcuts, extensions, autostart. It's not a general Linux sysadmin skill (services, users, networking, logs); pair it with something like [linux-sysadmin-skills](https://github.com/HermeticOrmus/linux-sysadmin-skills) if you want that too.

## Contributing

Issues and PRs welcome, especially:
- Ubuntu 24.04+/GNOME 46+ specifics if something here is stale
- Additional common workflows (e.g. Nemo/other file managers, more extension patterns)
- Corrections to any command that's drifted with a GNOME/Nautilus release

## License

MIT — see [LICENSE](LICENSE).
