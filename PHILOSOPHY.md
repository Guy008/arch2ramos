# Philosophy — what arch2ramos is, and what it isn't

This is a design document, not marketing. It explains the choices behind
the project so future contributors don't unmake them by accident.

## The core insight

**A desktop OS does two things that fight each other:**
1. *Be customizable* — install packages, edit configs, accept change.
2. *Be reliable* — boot consistently, not break under casual use.

Traditional desktops (Windows, macOS, "any Linux desktop") trade off
toward customizability. The OS is mutable. Every install/uninstall/click
changes the system. Over months, drift accumulates. Things break in ways
that are hard to undo.

Immutable distros (NixOS, Silverblue, ChromeOS, SteamOS, Batocera) trade
off toward reliability. The base OS is fixed; customization goes
elsewhere (Nix store, Flatpaks, /home, overlay layers). But they pay for
reliability with friction — every change is a multi-step ritual.

**arch2ramos picks reliability via a different mechanism: dual-state boot.**

- **Disk mode** is fully mutable Arch. Install whatever, break whatever.
- **RAM modes** are immutable snapshots of disk mode at the moment of last build.

This gives you both — but the customization happens in one mode and the
reliability happens in another. The cost of an update is one reboot.

## Why RAM specifically (and not just A/B partitions)?

- A/B uses disk, which means **writes during boot**. We get zero writes.
- A/B requires N+N+N space for each version. We need only the current.
- A/B's atomic-switch model still leaves a writable rootfs at runtime.
  Ours is **read-only at the block level** (squashfs + overlayfs upper = tmpfs).
- A/B can't reset to "factory" mid-day. We reboot and we're new.
- A/B feels like an upgrade mechanism. Ours feels like a **state model**.

## What we won't do

| | Why not |
|---|---|
| Persistent overlay (writes survive reboot) | Defeats the "every reboot = clean" guarantee. The whole point. If you want persistence, that's what `/home` is for, or reboot to disk mode. |
| In-RAM-mode package install | Same reason. Plus pacman on overlayfs has historically been buggy. Reboot to disk mode to install. |
| Custom desktop environment | The market is saturated. Hyprland, KDE, GNOME, sway exist. We *choose* not to compete there. Our value-add is below the DE. |
| Replace GRUB | GRUB works. systemd-boot works. We use what's already there. Replacing it would be busywork. |
| Tight Steam/Valve coupling | SteamOS already exists if you want SteamOS. We use gamescope+Steam because they're best-in-class, but ES and cage modes don't depend on Steam at all. |
| Hide complexity from advanced users | Disk mode is just Arch. No magic. `pacman -Syu` works normally. If you know Arch, you know us. |
| Curate the AUR / "store" | Arch users use pacman + AUR. We'd be reinventing yay for no reason. |
| Be all distros to all people | We target **desktops with ≥16 GB RAM and the specific audiences in [AUDIENCES.md]**. Generic Linux for laptops? Use Bazzite. Server Linux? Use Debian. Workstation? Use Fedora. |

## What we *do*

| | Why |
|---|---|
| Multi-mode boot | The same hardware serves different jobs (gaming console, kiosk, dev workstation, media center) without imaging different disks. |
| RAM-only root | Bulletproofs the OS. Free SSD lifespan. Eliminates "OS drift" forever. |
| Re-use proven mechanisms | `mkinitcpio-archiso`, `squashfs`, `overlayfs`, `gamescope`, `cage`, `hyprland`, ES — all existing, all stable. **We're the wiring, not the components.** |
| Hardware-aware defaults | simpledrm unbind for hybrid GPU. setpriv ambient-cap drop for bwrap. Things we hit and solved that nobody documented well. |
| User-data isolation | `/home` is always the writable disk mount. The OS *can't* corrupt your data, because the OS can't write back. |

## The "launcher is the environment" insight

We started with: ES is a launcher.
We ended with: **ES is the desktop environment for the kiosk/console use case.**

What's a "desktop environment"? At its core it's:
1. A way to launch apps.
2. A way to switch between them.
3. A way to configure the system.

For a TV / console / kiosk audience:
- Launching: ES menu does this better than start menus.
- Switching: irrelevant. One thing at a time, full screen.
- Configuring: a small set of menus (volume, network, brightness) that
  ES already provides hooks for.

So we don't need Plasma. We don't need GNOME. We **don't need a window
manager at all** beyond gamescope-as-compositor. ES + runer.sh +
arch2ram-launch is a complete environment for the target audience.

For users who want windowing (dev work, multi-tasking), the
`hyprland` mode is still there. Two answers, picked at GRUB.

## The "grandma can use it" test

The kiosk + ES boot modes pass this test:
- No window manager → no Alt+Tab confusion.
- No file manager visible by default → no accidental deletion.
- Single carousel of giant icons → universally legible.
- Power button = shutdown menu, no surprises.
- Crash? Reboot fixes everything.

This is what "advanced user accessibility" looks like: making the
machine simple enough for non-technical users while still being a full
Linux box under the hood.

## The AI integration future

The natural extension: an ES menu item that opens an AI prompt. User
asks "play Star Wars", the AI translates to: find the movie file → pipe
to mpv → launch via runer.sh. User asks "install GIMP", AI translates:
reboot to disk mode (via grub-reboot) → schedule pacman -S gimp +
arch2ram-create → reboot back.

This collapses the gap between "non-technical user" and "advanced
Linux". The user speaks intent; the launcher dispatches commands. The
OS underneath is still full Arch — anyone who wants to bypass the AI
and use a terminal can.

We haven't built this yet. We're laying the foundation: a dispatcher,
a menu, a known set of capabilities. The AI is a button away.

## What this is *not* a copy of

- **Batocera** — we share RAM-immutability and ES, but they target
  low-end ARM/x86 boxes for retro gaming. We target desktops with
  serious hardware for general computing + gaming + media.
- **SteamOS** — they're A/B-partition with Plasma in desktop mode.
  We're RAM-only with optional Plasma (and we recommend hyprland).
- **Bazzite** — they're rpm-ostree, mutable system writes during ops.
  We're zero-write at runtime.
- **NixOS** — they're declarative-everything, slow rebuild cycle. We
  embrace imperative Arch + a single squashfs rebuild.
- **ChromeOS** — they're Google-locked, narrow application set. We're
  fully open, full Arch ecosystem.

---

If a feature request would weaken any of "RAM-only", "zero-write at
runtime", "multi-mode boot", or "no DE required" — it goes against the
philosophy and gets a polite no. Anything else is fair game.
