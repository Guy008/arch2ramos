# arch2ramos EmulationStation integration

Reuses ES's "system" abstraction as **categories** (not emulators).
Each category points to a directory; every executable in that dir
becomes a "game" in ES. All categories funnel through a single
launcher script ([runer.sh](https://github.com/Guy008/...) by default)
that sets GPU/CPU/Vulkan optimization env vars before exec.

## Layout

```
/etc/emulationstation/es_systems.cfg   ← shipped here, system-wide
/home/$USER/.emulationstation/         ← user-specific (themes, settings)
/home/$USER/ES/                        ← category dirs
   ├── games/    ← Steam wrapper, native games, etc.
   ├── apps/     ← Browsers, players, file manager wrappers
   └── tools/    ← Utility wrappers
```

## How to add an entry

Create a `.sh` wrapper in the matching category dir. Example:

```bash
# /home/Guy008/ES/apps/calc.sh
#!/bin/bash
exec /usr/bin/kcalc "$@"
```

`chmod +x` and it'll appear in ES on next launch.

## Why /etc/emulationstation when /home/$USER/.emulationstation works?

Because **/home is mounted from disk in RAM mode**, but `/etc/emulationstation/`
is part of the squashfs — so fresh installs of arch2ramos get the
configuration automatically without needing to copy dotfiles.

The user-side `~/.emulationstation/` then layers on top with themes,
favorites, etc.
