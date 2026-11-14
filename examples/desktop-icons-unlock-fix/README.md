# Desktop icons vanish after screen unlock (GNOME 42 / Wayland)

A worked example of diagnosing and fixing a GNOME problem with this skill. Tested on Ubuntu 22.04.5, GNOME Shell 42.9, Nautilus 42.6, Wayland, single HDMI monitor.

## Symptom

Every so often all desktop icons disappear. Logging out and back in brings them back.

## Cause

Ubuntu's desktop icons come from the **Desktop Icons NG (DING)** extension (`ding@rastersoft.com`), which draws them from a separate `gjs` process. GNOME 42 turns extensions off while the screen is locked and back on at unlock, so DING restarts on every unlock.

Sometimes it restarts before the display has finished waking. With no valid monitor available, the user journal shows a burst of:

```
DING: (gjs:NNNN): Gdk-CRITICAL **: gdk_monitor_get_scale_factor: assertion 'GDK_IS_MONITOR (monitor)' failed
DING: Not enough space to add icons
```

DING then decides there's no room for icons and doesn't recover when the monitor comes back.

On the test machine, every `Not enough space to add icons` in a 30-day window came within a second or two of a screen unlock (`gkr-pam: unlocked login keyring`). It happened on about 30% of unlocks (41 failures in 143 unlocks), which fits a timing race.

To check whether you have the same problem:

```bash
journalctl --no-pager -o short-iso --since "-30 days" | grep -E 'gkr-pam: unlocked login keyring|DING: Not enough space'
```

If `DING: Not enough space` lines keep appearing right after `unlocked` lines, this fix applies.

## The fix

Restarting the extension brings the icons back without logging out. This folder automates that.

| File | Install to | Purpose |
|---|---|---|
| `bin/restart-desktop-icons` | `~/.local/bin/` | Turns DING off and back on. Run it by hand or from a keyboard shortcut. |
| `bin/desktop-icons-unlock-watcher` | `~/.local/bin/` | Listens for `org.gnome.ScreenSaver.ActiveChanged(false)` (unlock), waits 4 s, and checks the journal for `Not enough space to add icons` since the unlock. Only if that's there does it run the restart script, up to 3 times. Normal unlocks are left alone, so there's no icon flicker. Each restart is logged with the tag `desktop-icons-watcher`. |
| `autostart/desktop-icons-unlock-watcher.desktop` | `~/.config/autostart/` | Starts the watcher at login. |

## Install

```bash
install -m 755 bin/restart-desktop-icons bin/desktop-icons-unlock-watcher ~/.local/bin/
mkdir -p ~/.config/autostart
cp autostart/desktop-icons-unlock-watcher.desktop ~/.config/autostart/
```

Start the watcher now, without logging out:

```bash
setsid -f ~/.local/bin/desktop-icons-unlock-watcher >/dev/null 2>&1 </dev/null
```

Optional manual shortcut (Super+Alt+D). This **overwrites** your custom keybinding list, so if you already have custom shortcuts, append the path to the existing array and use the next free `customN`:

```bash
P=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/
S=org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:$P
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['$P']"
gsettings set $S name 'Restart desktop icons'
gsettings set $S command "$HOME/.local/bin/restart-desktop-icons"
gsettings set $S binding '<Super><Alt>d'
```

## Checking it worked

```bash
journalctl -t desktop-icons-watcher --since today
```

## Uninstall

```bash
pkill -f desktop-icons-unlock-watcher
rm ~/.local/bin/desktop-icons-unlock-watcher ~/.local/bin/restart-desktop-icons
rm ~/.config/autostart/desktop-icons-unlock-watcher.desktop
```

Remove the shortcut under Settings → Keyboard → Custom Shortcuts.

## Notes

- This is a workaround, not a fix for DING itself. Newer DING releases (on Ubuntu 24.04+) may not have the race.
- The watcher depends on DING's exact log message. If a DING update changes the wording, the watcher stops finding failures, and the manual shortcut still works.
