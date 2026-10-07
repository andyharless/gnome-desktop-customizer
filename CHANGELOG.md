# Changelog

All notable changes to this skill are recorded here. The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repo has no version numbers yet, so entries are grouped by date.

## 2026-10-07 — Prompt audit

### Fixed
- **SKILL.md:** The GNOME 46+ overview said the nautilus-python API was "largely the same", contradicting the `MenuProvider` step, which says the signatures changed in Nautilus 43. The overview now points to that step.

## 2026-09-27 — Accuracy review

A review pass over the initial files, done with Claude Code. Detailed rationale for each change: [docs/2026-09-27-review.md](docs/2026-09-27-review.md).

### Fixed
- **SKILL.md:** The nautilus-python `MenuProvider` signatures were reversed between GNOME 42 and 43+. Rewrote the step to show both versions and added the required `gi.require_version(...)` call.
- **SKILL.md:** Replaced the `nautilus --no-desktop` debugging command (that flag no longer exists) with `nautilus -q; nautilus`.
- **SKILL.md:** Changed the example custom shortcut from `<Control><Alt>t`, which is Ubuntu's default terminal shortcut, to `<Super><Alt>m`.
- **SKILL.md:** GNOME Tweaks → Startup Applications is now described as GNOME 42 only, with Ubuntu's `gnome-startup-applications` app given as the alternative.
- **README.md:** Fixed the broken Agent Skills announcement link.
- **SKILL.md, README.md:** The "files are only written under `~/.config`, `~/.local/share`, `~/Templates`" statement now mentions the backup files that are written to `~`.

### Added
- **README.md:** Notice that the repo was written with Claude and has not been fully human-reviewed.
- **SKILL.md:** Notes that menu-item icons aren't shown in GTK4 Nautilus, and that a signature mismatch makes an entry fail silently.
- **SKILL.md:** How to check for a shortcut conflict before binding, and to use the next unused `customN` path.
- **SKILL.md:** `gnome-extensions install <file.zip>` and `list --enabled`; the `gnome-browser-connector` / `chrome-gnome-shell` package names; the log out/in requirement on Wayland for newly installed extensions.

### Changed
- **README.md:** Replaced the vague "device-bridge/remote-computer" remark with a plainer note about remote/SSH sessions.

## 2026-09-27 — Initial commit

- Initial `SKILL.md`, `README.md`, `LICENSE` (MIT), and `.gitignore`, written in a Claude web session (Sonnet) and pushed manually.
