# arch2ramos

> **A bulletproof, immutable Arch-based Linux distribution that lives entirely in RAM.
> Boot it, use it, reboot — it's perfectly fresh again.**

```
┌─────────────────────────────────────────────────────────────────┐
│  arch2ramos  =  Arch Linux  +  RAM-only root  +  pick-your-UI   │
│                                                                  │
│  • Compressed system snapshot in RAM (zstd)                     │
│  • Zero writes to root disk after boot                          │
│  • Every reboot = clean install                                 │
│  • Pick a UI at GRUB: gamescope · ES · cage · hyprland · TTY    │
│  • Same hardware → 90% of what Batocera + SteamOS + ChromeOS do │
└─────────────────────────────────────────────────────────────────┘
```

It's not a tool — it's a **distribution**. You install Arch your way, you
configure it your way, then `arch2ramos` packages it into an immutable
RAM-bootable image you can use as your daily driver.

---

## Why?

Different problems, one architecture solves all of them:

| Audience | Why arch2ramos? |
|---|---|
| **Gamers** | SteamOS-style experience on YOUR hardware. gamescope+Steam Big Picture as default boot. Native 4K, FSR, MangoHud built-in. |
| **Retro enthusiasts** | EmulationStation launcher with 19 systems out of the box (NES → PS3 via wine/Proton for everything Windows runs). RAM-immutable like Batocera. |
| **Media center / TV PC** | Boots fast, doesn't wear the SSD, doesn't break when family clicks things. Reboot = factory reset. |
| **Hackers / security researchers** | Run forensic tools, install random AUR packages, browse anything — reboot wipes it all. Persistent forensic evidence stays on `/home`; the OS itself is read-only. |
| **Internet cafés / public computers** | Single-window kiosk mode, no Alt+F4, no window manager to confuse non-technical users. Customers can't break the OS. Every shift starts on a clean machine. |
| **Schools / labs** | One known-good image. Every student boots into an identical environment. Updates roll out by rebuilding the image, not by chasing individual machines. |

---

## How it works (one paragraph)

GRUB loads the kernel + an [archiso](https://wiki.archlinux.org/title/Archiso)-style
initramfs that finds your squashfs by UUID, copies it into a tmpfs
(`copytoram=y`), loop-mounts it, layers a writable tmpfs on top via
**overlayfs**, then `switch_root`s into the result. systemd boots
normally on top of that — **but every write goes to the in-RAM
overlay and dies at the next reboot**. Your `/home` is a separate
mount from the real disk, so personal data survives.

This is the same mechanism the official **Arch Linux installer ISO**
uses to boot. We re-use it for your installed system.

```
GRUB  →  kernel + initramfs-arch2ram.img
            ↓
         archiso hook
            ├─ find partition by UUID
            ├─ copy airootfs.sfs into tmpfs       (~1.5s on NVMe Gen4)
            ├─ loop-mount squashfs (read-only)
            ├─ tmpfs as overlay writes layer
            └─ overlayfs: lower=squashfs, upper=tmpfs → /new_root
            ↓
         switch_root → systemd → your boot mode (see below)
```

---

## Five ways to boot — pick at GRUB

| GRUB entry | What launches | Use case |
|---|---|---|
| **gamescope** ⭐ default | gamescope compositor + Steam Big Picture | Daily driver, gaming console, media center |
| **EmulationStation** | gamescope + ES + 19-system carousel + runer.sh | Batocera-style retro & native launcher |
| **kiosk** | cage (single-window compositor) + konsole | One-app TV terminal, vending machine, signage |
| **hyprland** | Hyprland tiling Wayland desktop | "Normal" desktop with workspaces & windows |
| **debug TTY** | `multi-user.target` only | SSH, system rescue, server-mode |

Plus a **disk** entry that boots your normal writable Arch (where you run
updates, edit configs, then `arch2ram-update` to rebuild the image).

---

## Architecture comparison

|  | arch2ramos | Batocera | SteamOS | Bazzite | ChromeOS |
|---|---|---|---|---|---|
| Base | Arch | Buildroot | Arch | Fedora | Gentoo |
| Immutable | ✓ (RAM) | ✓ (RAM overlay) | ✓ (A/B partitions) | ✓ (rpm-ostree) | ✓ (rootfs verity) |
| User-installable packages | ✓ via disk-mode + rebuild | ✗ | partial (Flatpak) | ✓ (Flatpak/distrobox) | partial (Crostini) |
| Gaming-first | ✓ (gamescope+Steam) | ✓ (RetroArch+Batocera UI) | ✓ (gamescope+Steam) | ✓ (Steam stack) | ✗ |
| Multi-mode boot | ✓ (5 modes) | ✗ (single ES boot) | ✗ | ✗ | ✗ |
| Runs any Linux app | ✓ (Arch ecosystem) | partial (locked emulators) | partial (locked UI) | ✓ | partial |
| Runs Windows games | ✓ (Wine/Proton/Lutris) | ✓ (Wine) | ✓ (Proton) | ✓ (Proton) | ✗ |
| Hardware target | desktop with ≥16 GB RAM | low-end ARM/x86 | Steam Deck | desktop | Chromebook |
| Update model | reboot to disk + `arch2ram-update` | overlay-persistent in-place | A/B swap | rpm-ostree atomic | Google push |
| Zero disk writes | ✓ | ~zero (overlay→tmpfs) | partial | ✗ (rpm-ostree writes) | partial |

What's **unique** to arch2ramos vs everyone else: the combination of full
Arch package ecosystem **+** RAM immutability **+** boot-mode selector
at GRUB. Bazzite is closest in spirit but writes to disk constantly.

---

## Install (5 minutes on a fresh Arch)

```bash
git clone https://github.com/Guy008/arch2ramos.git
cd arch2ramos

# 1. Install the boot scaffolding (deps, initramfs, GRUB entries):
sudo scripts/arch2ram-install --with-emulationstation
   # adds:  gamescope · cage · konsole · hyprland (if installed)
   # adds:  EmulationStation from AUR (~149 MB)
   # auto-discovers your installed apps + games (/usr/share/applications,
   # /media/G/{Linux,Windows}/) and creates ES wrappers
   # writes 5 RAM-mode GRUB entries + 2 disk-mode entries

# 2. Build the squashfs (3–8 minutes depending on system size):
sudo scripts/arch2ram-create

# 3. Reboot, pick a mode in GRUB.
reboot
```

### Update cycle

```bash
# Boot the disk-mode "Arch Linux (disk, UKI — for updates)" entry, then:
sudo arch2ram-update          # pacman -Syu + mkinitcpio + arch2ram-create
reboot                        # back into your chosen RAM mode
```

---

## Boot modes in detail

### `gamescope` (default)
gamescope is Valve's compositor (the one in the Steam Deck). Owns DRM
directly — no Plasma, no GNOME, no hyprland underneath. We launch it
with `setpriv --ambient-caps '-all' gamescope ... -- steam -bigpicture`.
You get Steam Big Picture in 4K, with adaptive sync, MangoHud overlay,
and Steam handles all input device mapping.

Override: `arch2ram.gs.res=1920x1080`, `arch2ram.gs.rate=120`,
`arch2ram.gs.launcher=/path/to/your-thing` on the kernel cmdline.

### `EmulationStation`
Same gamescope compositor — but instead of Steam BPM, runs **ES** as
the launcher. ES shows 19 systems (apps · linux · windows · steam ·
nes · snes · n64 · gb · gbc · gba · nds · megadrive · mastersystem ·
dreamcast · psx · ps2 · ps3 · psp · dos · arcade · ...). Each system
funnels through `arch2ram-launch SYSTEM ROM`, which picks the right
emulator/runner and wraps with `runer.sh` for max GPU acceleration.

### `kiosk` (cage)
Pure single-window mode. Launches `konsole` fullscreen with a
Hebrew-BiDi profile at 36pt. Replace with any binary via launcher
override. **Perfect for**: internet cafés, vending machines, classroom
display walls, signage. Non-technical users cannot escape the running app.

### `hyprland`
The user's existing hyprland config is sourced; we add a wrapper that
adds a connector-name override for RAM-mode HDMI labeling. Multi-window
tiling Wayland desktop with bar, tray, workspaces.

### `debug TTY`
`multi-user.target` only — no graphics. For SSH access, system rescue,
or running headless workloads.

---

## What's in the image

The squashfs contains everything from `/`, minus:

**Always excluded** (path-based):
- `/home` (mounted from disk in RAM mode — your data persists)
- `/proc`, `/sys`, `/dev`, `/run`, `/tmp`, `/mnt`, `/media` (runtime)
- `/var/cache/pacman/pkg`, `/var/log`, `/var/tmp`, `/var/lib/systemd/coredump`
- `/usr/lib/debug`, `/usr/share/locale`, `/usr/share/doc`, `/usr/share/man`
- `/opt/cuda`, `/opt/android-studio`, `/opt/rocm` (heavy, rarely used)

**Optionally excluded** (per-package, via `/etc/arch2ram/strip-packages.conf`):
Whole desktop environments (Plasma, GNOME, Cinnamon), display managers
(GDM, SDDM, LightDM), AUR helpers (yay, paru), build tools (gcc, cmake,
ninja), kernel headers — anything you don't need at runtime. **Disk
copies stay intact** — only the squashfs gets the slimmer view.
Sample saves: ~2-3 GB before compression → ~800 MB-1 GB on disk.

---

## Hardware requirements

- **CPU**: any x86_64 from the last decade
- **RAM**: minimum 8 GB usable (squashfs ~5 GB compressed + ~3 GB
  working). 16 GB recommended, 32 GB ideal for gaming + apps
- **GPU**: any KMS-capable card. **Intel iGPU + AMD dGPU hybrid is
  the explicit primary target** (the project's simpledrm-unbind
  workaround makes this combo work where SteamOS struggles).
  Pure AMD and pure Intel also fine. Nvidia partial (needs `nouveau`;
  proprietary driver compatibility not yet tested).
- **Storage**: ~6 GB on the boot partition for the squashfs +
  initramfs. NVMe Gen3+ recommended (RAM load takes 1-2 seconds).
- **Bootloader**: GRUB (UEFI or BIOS). systemd-boot support: untested.

---

## Project layout

```
arch2ramos/
├── README.md                       ← you are here
├── PHILOSOPHY.md                   why this exists, what we won't do
├── AUDIENCES.md                    7 user types this is built for
├── MORNING-NOTES.md                latest session change log
├── strip-packages.conf.example     opt-in slimmer image
│
├── scripts/
│   ├── arch2ram-install            one-time setup, picks boot modes
│   ├── arch2ram-create             build the squashfs from current /
│   ├── arch2ram-update             update cycle (pacman + create)
│   ├── arch2ram-uninstall          full revert (incl. --purge)
│   │
│   ├── arch2ram-drm-fixup          simpledrm unbind (hybrid GPU fix)
│   ├── arch2ram-gamescope          gamescope + Steam BPM launcher
│   ├── arch2ram-emulationstation   gamescope + ES launcher
│   ├── arch2ram-kiosk              cage + konsole launcher
│   ├── arch2ram-hyprland           hyprland launcher
│   ├── arch2ram-launch             ES system dispatcher (19 systems)
│   └── arch2ram-es-discover        scan /usr/share/applications + /media/G
│
├── mkinitcpio/
│   ├── arch2ram.conf               hooks + compression
│   └── arch2ram.preset             pacman-hook-compatible preset
│
├── systemd/                        4 services (drm-fixup + per-mode launcher)
├── env/                            user environment generators
├── konsole/                        kiosk profile (Hebrew BiDi, 36pt for 4K TV)
├── hyprland/                       wrapper config for RAM-mode HDMI quirk
├── emulationstation/               es_systems.cfg + README
├── grub/                           menuentry templates
└── docs/                           boot-flow, troubleshooting
```

---

## License & status

Personal-use license. No warranty. Pull requests welcome but the
project is opinionated — driven by what makes it good for the
audiences listed above, not by "make it generic for everyone".

For the actual design principles and what we DON'T do, see
**[PHILOSOPHY.md](PHILOSOPHY.md)**.

For who this is for in concrete detail, see
**[AUDIENCES.md](AUDIENCES.md)**.
