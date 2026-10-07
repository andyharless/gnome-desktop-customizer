---
name: gnome-desktop-customizer
description: Use when customizing the GNOME desktop or Nautilus/Files file manager on Ubuntu — right-click context menu additions, gsettings/dconf tweaks, keyboard shortcuts, GNOME Shell extensions, autostart apps. Covers GNOME 42 (Ubuntu 22.04) and GNOME 46+ (Ubuntu 24.04+).
---

# GNOME Desktop Customizer (Ubuntu)

This skill turns "I want GNOME/Nautilus to do X" requests into concrete, safe changes on Ubuntu, without the user having to search docs or hand-roll config themselves. It targets GNOME 42 (Ubuntu 22.04 LTS) and GNOME 46+ (Ubuntu 24.04 LTS and newer). The user may be on either version — check before doing anything version-sensitive.

## First step: identify the environment

Run before making changes that could differ by version:
```
lsb_release -ds
gnome-shell --version
nautilus --version
```
GNOME 42 (22.04): Nautilus is still GTK3-based. `nautilus-python` extensions use the older `Nautilus.MenuProvider` API and are the standard way to add custom right-click actions.
GNOME 46+ (24.04+): Nautilus is GTK4-based. `nautilus-python` still works, but its `MenuProvider` method signatures changed in Nautilus 43 (see the nautilus-python workflow below), and packaging/paths can differ; verify `nautilus-python` is available via `apt-cache policy python3-nautilus` before assuming it's installable the same way. When advice differs by version, say so explicitly rather than giving one silently-version-specific answer.

## Safety boundaries

- Everything in this skill lives under `~/.config`, `~/.local/share`, or `~/Templates` (plus backup files written to `~`) — user-space, no `sudo` needed except to `apt install` a helper package (nautilus-python, dconf-editor, gnome-tweaks, gnome-shell-extension-manager). Ask before running any `sudo apt install`, name the package and why.
- Never hand-edit the dconf database file directly. Always go through `gsettings` or `dconf write`/`dconf load`.
- Before changing a `gsettings`/`dconf` key that isn't trivially reversible, read and record the current value first (`gsettings get <schema> <key>`), so it can be restored:
  ```
  gsettings get org.gnome.example.schema some-key
  ```
  For a broader change (several keys under one path), back up first:
  ```
  dconf dump /org/gnome/some/path/ > ~/gnome-backup-$(date +%Y%m%d)-somepath.conf
  # restore with: dconf load /org/gnome/some/path/ < backup-file
  ```
- After a change, verify it took effect with `gsettings get` (or by observing the UI) rather than assuming the `set` succeeded.
- Nautilus changes need `nautilus -q` (quits all Nautilus windows/process; it restarts on next open) to take effect. GNOME Shell extension/keybinding changes usually apply live; if not, Alt+F2 → `r` → Enter reloads GNOME Shell (Xorg sessions only — on Wayland the user must log out and back in). Newly *installed* (as opposed to enabled/disabled) extensions also need a log out/in on Wayland before GNOME Shell sees them.
- Don't touch `/usr/share/nautilus-python` or other system dirs; use the user-space extension directories below.

## Workflow: add "New File"/"New Document" to the right-click menu

This is GNOME's built-in Templates feature — no extension needed, works identically on GNOME 42 and 46+.

1. Confirm the Templates directory is registered:
   ```
   grep XDG_TEMPLATES_DIR ~/.config/user-dirs.dirs
   ```
   Should show `XDG_TEMPLATES_DIR="$HOME/Templates"` (or a translated name if the system locale isn't English). If the line is missing or the folder doesn't exist, create both:
   ```
   mkdir -p ~/Templates
   xdg-user-dirs-update
   ```
2. Create an empty file in `~/Templates` named whatever should appear in the menu, e.g.:
   ```
   touch ~/Templates/'Empty File.txt'
   ```
   Any file works as a template — a `.txt`, a `.md`, a pre-filled script with a shebang, a blank `.docx`, etc. The filename becomes the submenu entry under New Document.
3. `nautilus -q` if a Nautilus window is already open, then right-click in any folder → New Document → the new entry should be there.
4. To add more template types, repeat step 2 with different filenames/extensions in `~/Templates` (subfolders are also supported for grouping).

## Workflow: quick one-off right-click actions (Nautilus Scripts)

Simplest option for "run this command on the selected file(s)" style actions — no extension framework needed.

1. `mkdir -p ~/.local/share/nautilus/scripts`
2. Add an executable script there (`chmod +x`). Nautilus passes context via env vars, notably `NAUTILUS_SCRIPT_SELECTED_FILE_PATHS` (newline-separated absolute paths of the current selection) and `NAUTILUS_SCRIPT_CURRENT_URI`.
3. The script's filename becomes the menu label, shown under a "Scripts" submenu on right-click.
4. `nautilus -q` to pick it up.

Good for: quick conversions, hashing, opening a terminal at a path, batch renames. Not good for: needing a live/dynamic label, icons, or conditional visibility — use nautilus-python for that.

## Workflow: real context-menu actions (nautilus-python)

For entries that need conditions (only show for certain file types), icons, or integrate more deeply.

1. Install the binding: `sudo apt install python3-nautilus` (ask first — this is the one step needing sudo).
2. Extension file goes in `~/.local/share/nautilus-python/extensions/your_extension.py`.
3. The `MenuProvider` API differs between versions — check which `python3-nautilus` is installed (`apt-cache policy python3-nautilus`) and use the matching shape:
   - **GNOME 42 / Nautilus 42** (nautilus-python 1.x, `Nautilus` typelib 3.0): methods take a leading `window` argument — `get_file_items(self, window, files)`, `get_background_items(self, window, current_folder)`.
   - **GNOME 43+ / Nautilus 43+** (nautilus-python 4.x, `Nautilus` typelib 4.0): the `window` argument was removed — `get_file_items(self, files)`, `get_background_items(self, current_folder)`. Menu item icons are no longer displayed.

   Minimal shape for GNOME 43+ (for GNOME 42, change the version to `"3.0"` and add the `window` parameter):
   ```python
   import gi
   gi.require_version("Nautilus", "4.0")
   from gi.repository import Nautilus, GObject

   class ExampleExtension(GObject.GObject, Nautilus.MenuProvider):
       def get_file_items(self, files):
           if len(files) != 1:
               return []
           item = Nautilus.MenuItem(
               name="Example::action",
               label="Do the thing",
               tip="Runs the custom action",
           )
           item.connect("activate", self.on_activate, files[0])
           return [item]

       def on_activate(self, menu, file_obj):
           path = file_obj.get_location().get_path()
           # do the actual work here
   ```
   A signature mismatch doesn't produce a visible error in the UI — the entry just silently fails to appear — so this is the first thing to check. Installed examples (`/usr/share/doc/python3-nautilus/examples/`) show the correct signatures for the installed version.
4. `nautilus -q` to reload. If it still doesn't show up, run `nautilus -q; nautilus` from a terminal — Python exceptions from the extension are printed there.

## Workflow: gsettings/dconf tweaks (behavior, appearance, window management, etc.)

1. Find the right schema/key. If the user describes a Settings-app toggle, `gnome-tweaks` or `dconf-editor` (GUI, browsable, shows the schema path for any control) is the fastest way to locate the exact key — install with `sudo apt install dconf-editor gnome-tweaks` if not present.
2. Read current value, set new value, verify:
   ```
   gsettings get org.gnome.desktop.interface clock-show-seconds
   gsettings set org.gnome.desktop.interface clock-show-seconds true
   gsettings get org.gnome.desktop.interface clock-show-seconds
   ```
3. For custom keyboard shortcuts, the pattern is an array of paths plus per-path settings:
   ```
   gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings \
     "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"
   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'My Shortcut'
   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command '/path/to/script'
   gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding '<Super><Alt>m'
   ```
   If existing custom keybindings are already present, read the current array first and append rather than overwrite it (use the next unused `customN` path). Also pick a binding that isn't already taken — e.g. `<Control><Alt>t` is Ubuntu's default terminal shortcut; check with `gsettings list-recursively | grep -F '<binding>'`.

## Workflow: GNOME Shell extensions

- Prefer installing/managing through the GNOME Extensions app (`gnome-shell-extension-manager` on 22.04, built-in "Extensions" app on newer/some Ubuntu builds) or extensions.gnome.org (needs the browser extension plus the `gnome-browser-connector` package — `chrome-gnome-shell` on 22.04) over hand-copying extension dirs, unless the user wants a specific dev/unpublished extension.
- CLI alternative: `gnome-extensions install <file.zip>`, `gnome-extensions list --enabled`, `gnome-extensions enable <uuid>`, `gnome-extensions disable <uuid>`.
- GNOME 42 vs 46+ matters here: an extension's `metadata.json` declares supported shell versions, and many older extensions are not updated for 46. If the user is upgrading Ubuntu versions, flag that their current extensions may need updated equivalents post-upgrade.

## Workflow: autostart applications

Drop a `.desktop` file in `~/.config/autostart/`, e.g.:
```
[Desktop Entry]
Type=Application
Name=My Startup Task
Exec=/path/to/command
X-GNOME-Autostart-enabled=true
```
Or use the GUI: `gnome-tweaks` → Startup Applications on GNOME 42 (that panel was removed from newer GNOME Tweaks), or Ubuntu's standalone "Startup Applications" app (`gnome-startup-applications`) if installed.

## When something breaks

- Nautilus acting up after a change: `nautilus -q` and reopen.
- GNOME Shell acting up: Alt+F2 → `r` → Enter (Xorg only) or log out/in (Wayland — check current session type with `echo $XDG_SESSION_TYPE`).
- A dconf path went wrong and a backup was taken: `dconf load /org/gnome/some/path/ < backup-file`.
- Nuclear option for a schema tree back to defaults (explain this erases customization in that tree first): `dconf reset -f /org/gnome/desktop/`.
