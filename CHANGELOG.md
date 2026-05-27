# arch2ramos changelog

## v0.3 -- 2026-05-27 -- "no WM, no DE"

Built across four focused sprints after a deep dive into Batocera v43
and Batocera.PLUS. Theme: turn arch2ramos from a "RAM-boot tool" into
a launcher-first immutable distribution.

### Sprint 1 -- universal hotkeys

* `triggerhappy` runs at the evdev layer. Hotkeys work in EVERY boot
  mode (gamescope, ES, cage, hyprland, even raw TTY) without any WM
  or DE.
* `arch2ram-th-run` resolves the active graphical user via `loginctl`,
  injects DISPLAY/WAYLAND_DISPLAY/XAUTHORITY/HOME/XDG_RUNTIME_DIR/DBUS,
  drops privileges via `runuser`. Triggerhappy is root, commands are
  the user.
* Six helper scripts: `arch2ram-{mouse,screenshot,help,brightness,
  kill-foreground,th-run}`.
* Bindings: `Win+F1` home, `Win+F2` help, `Win+F3` mouse toggle,
  `Win+F4` konsole, `Win+F12` / `PrtSc` screenshot, `Win+ESC`
  kill-foreground, volume / brightness across both dedicated keys
  and `Win+arrows` fallback.

### Sprint 2 -- ES production wrapper + 307 controllers + kiosk lockdown

* `arch2ram-emulationstation` rewritten as an exit-loop launcher
  (Batocera pattern). Honors `/tmp/arch2ram.{shutdown,reboot,restart-es,
  switch-mode-X}` triggers between ES exits. Rapid-crash guard drops
  to a recovery konsole inside gamescope.
* `es_input.cfg` grew from 21 to 6190 lines: 307 controller mappings
  imported directly from upstream Batocera (Xbox, DualShock, 8BitDo,
  Switch Pro, etc.). Our PlayStation-style keyboard mapping (A=accept)
  preserved.
* `arch2ram-disablealtfn` (port of Batocera S33disablealtfn) strips
  Console_<N> bindings via `dumpkeys` -> `loadkeys`. Three safety
  guards prevent locking the disk-mode boot.

### Sprint 3 -- multi-GPU primary selection + settings store

* `arch2ram-pick-primary-gpu` (port of S30checkprime): enumerates PCI
  class 0x03xx devices, finds which one has a connected display, marks
  it primary. Writes `/etc/arch2ram/gpu.env`; NVIDIA case also writes
  `/etc/X11/xorg.conf.d/10-arch2ram-prime.conf`. Previous "VGA|3D"
  regex falsely matched any device with "3D" in its description (e.g.
  "SanDisk Ultra 3D NVMe") -- now filters strictly by PCI class.
* `arch2ram-settings` -- three-layer key-value store (factory defaults
  -> system override -> user override). Commands: get/set/unset/list/
  keys. Ships with 22 sensible defaults in
  `/etc/arch2ram/settings.defaults`.
* `docs/INIT-ORDER.md` -- documents the S05/S30/S31/S70 stage map
  with explicit Before=/After= reasoning per service.

### Sprint 4 -- single-entry GRUB + release polish

* `arch2ram-install` now defaults to `--grub-single`: one entry "Arch
  Linux from RAM" boots into `arch2ram.mode=$DEFAULT_MODE`, the
  boot-mode orchestrator picks the launcher. Mode switching happens
  in-session via ES Actions, not by rebooting. `--grub-multi` keeps
  the old behavior for testing.
* `arch2ram-boot-mode` handles `debug`/`recovery` mode (stays at
  multi-user.target -- the safety-net entry).
* Default mode flipped from `gamescope` to `emulationstation` to
  match the launcher-as-OS direction.

### Numbers

| Metric | v0.2 | v0.3 | delta |
|---|---|---|---|
| Scripts in `/usr/local/bin/` | 17 | 25 | +8 |
| Systemd units | 5 | 7 | +2 |
| GRUB entries | 5 | 2 | -3 |
| ES controller mappings | 1 (keyboard) | 307 | +306 |
| Boot modes (launcher services) | 4 | 4 | 0 (in-session switchable now) |

---

## v0.2 -- earlier overnight sessions

25 ES categories, ~2,330 auto-discovered entries, Hebrew MP3 tag
encoding fix, AI integration via claude-code CLI, 5 GRUB boot modes,
custom Carbon theme with emoji-SVG icons, music --by-artist mode,
news/dashboard/weather/recent categories, runer.sh wrap for all
launches. See MORNING-NOTES.md for details.
