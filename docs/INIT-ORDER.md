# arch2ramos init ordering

Reference for which arch2ramos services run when, and what they depend on.
Mirrors the Batocera /etc/init.d/ numbering pattern (S05 = very early,
S99 = very late) but expressed in systemd ordering directives.

## Stage map

```
boot
 │
 ├─ S05  arch2ram-pick-primary-gpu    Picks the GPU with a connected display.
 │       (After=systemd-udev-settle)  Writes /etc/arch2ram/gpu.env so later
 │       (Before=drm-fixup)           steps know which card is "real".
 │
 ├─ S05  arch2ram-drm-fixup           Unbinds simpledrm. Hybrid Intel+AMD
 │       (After=pick-primary-gpu)     systems need this before user-space
 │                                    grabs the framebuffer.
 │
 ├─ S30  arch2ram-disablealtfn        Strips Console_<N> bindings from
 │       (After=vconsole-setup)       loadkeys. Only runs when copytoram=y;
 │       (Before=getty.target)        disk mode is unaffected.
 │
 ├─ S31  arch2ram-boot-mode           Reads /proc/cmdline arch2ram.mode=X,
 │       (After=multi-user.target)    starts arch2ram-<X>.service.
 │
 └─ launcher (one of)
    ├─ arch2ram-gamescope.service       Conflicts= with the other three.
    ├─ arch2ram-emulationstation.service  In-session switching = systemctl
    ├─ arch2ram-hyprland.service         start <other>; the running one is
    └─ arch2ram-kiosk.service            stopped automatically.
```

## Late services (S70+)

Not yet implemented, slated for future work:

```
S70  arch2ram-fans          fan PWM ramp + thermal-throttle bypass
S91  arch2ram-samba         auto-share /media drives over SMB
S95  arch2ram-wifi-auto     reconnect remembered networks
```

## Triggerhappy

`triggerhappy.service` (the upstream unit) starts at multi-user.target.
Our `/etc/systemd/system/triggerhappy.service.d/override.conf` only
changes the user (root, not nobody) so commands can `runuser` into the
user session. Trigger config is in `/etc/triggerhappy/triggers.d/`.

## Settings access pattern

The boot path expects every service to read settings via:

```
arch2ram-settings get <key> [default]
```

In particular, `arch2ram-pick-primary-gpu` writes its choice to
`/etc/arch2ram/gpu.env`; later units source that file rather than
running `lspci` again.

## Trigger files (runtime, not init)

When ES wants to act outside its own process, it writes a flag file
into `/tmp/`:

```
arch2ram.shutdown          poweroff after ES exits cleanly
arch2ram.reboot            reboot after ES exits cleanly
arch2ram.restart-es        relaunch ES (e.g. after theme change)
arch2ram.switch-mode-X     switch to launcher X
```

The `arch2ram-emulationstation` wrapper polls these after every ES
exit. Same pattern as Batocera's `emulationstation-standalone`.
