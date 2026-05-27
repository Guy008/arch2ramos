# Audiences — who is this built for?

arch2ramos is opinionated. It's *not* "Linux for everyone". It's a
strong fit for these specific audiences, where it solves real problems
that existing distros don't:

---

## 1. The PC Gamer who wants a console experience

**Problem with Windows**: 30 GB of telemetry/Defender/updates running
on the same machine you want to play on. Random "Update is restarting
your PC" mid-game. SSD lifetime burned on `RuntimeBroker.exe`.

**Problem with Steam Deck**: locked to Valve's hardware. 16 GB RAM cap.
Smaller GPU than your desktop.

**arch2ramos**: SteamOS-style boot (gamescope + Steam Big Picture) on
**your** hardware. 4K, your RTX/AMD card, your 32 GB RAM, native FSR/VRR.
Zero background services. Reboot resets everything if a game corrupts
something. Lutris exports for Windows games. **Best of both worlds.**

---

## 2. The Retro Gamer (Batocera/Recalbox user, but wants more)

**Problem with Batocera**: locked to the Batocera build system. Custom
emulators are hard. Buildroot is friction. Limited general-purpose use.

**Problem with vanilla RetroArch on a desktop**: no immutability, no
clean kiosk mode, no integrated UI for non-gaming things.

**arch2ramos**: full Arch, install any emulator from `pacman -S` or
AUR. 19+ systems out of the box. EmulationStation mode for the
Batocera-style carousel. ROMs live on a separate disk (`/media/R/`)
that arch2ramos doesn't touch. **Generalized Batocera with infinite
extensibility.**

---

## 3. The Media Center owner (HTPC in the living room)

**Problem with Windows/macOS HTPC**: heavy, slow, breaks, eats SSD.

**Problem with LibreELEC / OSMC**: locked to Kodi. Hard to add a
browser or a random game.

**arch2ramos**: boot to ES (or gamescope), open mpv or Kodi or Plex
or Stremio or a browser as a "game" in any category. 4K native, HDR
via gamescope. Zero disk wear (TV computer lasts 10 years). **TV
computer that does everything, doesn't break, doesn't update during
movies.**

---

## 4. The Internet Café / Public Computer Operator

**Problem with Windows public terminals**: virus magnet, users break
configs, IT spends time re-imaging machines, expensive software
licenses, anti-cheat on games requires admin.

**Problem with traditional "kiosk Linux"**: limited browser-only, or
heavy stack with usable DE that customers can break.

**arch2ramos kiosk mode**: cage + single app (browser, Steam, ES). No
window manager — **Alt+F4 does nothing**. No file manager visible. No
"open new window" by right-click. Crash? Reboot — clean machine, no
trace of previous customer. Every shift is factory-fresh.

For variations: ES mode lets the operator give customers a CHOICE of
games/apps but still no shell access. **Magic Card replacement
without buying a Magic Card** — and the disk doesn't wear.

---

## 5. The Senior Citizen (grandma's computer)

**Problem with Windows for non-tech users**: dialog boxes everywhere,
fake virus popups, "your Adobe needs update", random apps install
themselves, files end up in mysterious folders.

**Problem with iPad**: locked ecosystem, hard to add custom things,
expensive.

**arch2ramos ES mode**: giant icons, one carousel, one click =
launch. No file system to navigate, no settings to misclick. **Family
member sets up the categories** (Movies, Photos, Skype, Email), grandma
clicks pictures. AI integration (future) can voice-translate "show me
photos from last summer" → mpv with right files. **Senior-friendly
Linux that doesn't sacrifice power.**

---

## 6. The Hacker / Security Researcher

**Problem with VMs**: heavy, slow, weird hardware.

**Problem with persistent Linux**: traces of what you did stay forever.

**arch2ramos**: install whatever forensic / pentesting / experimental
tools you want from AUR. Run anything. **Reboot = bare machine.**
Useful evidence stays on `/home`; everything else evaporates.
Browser history, AI prompts, downloaded scripts, malware samples — all
gone. **Best ephemeral environment short of Tails, with full Arch
toolchain.**

---

## 7. The Classroom / Computer Lab

**Problem with traditional labs**: 30 machines drift independently.
"It works on the teacher's machine but not student 12's." Half a day
of IT for imaging.

**arch2ramos**: one master Arch install on a build machine. `arch2ram-create`
once. Copy the squashfs to all student machines. They all boot
identical environments every morning. Update = rebuild on master, sync
to fleet, done. **Reproducible lab without imaging tools.**

---

## Who arch2ramos is NOT for

- **Laptops where battery matters most**: RAM mode keeps the
  squashfs decompressed = more RAM = slightly more power. Disk
  mode is better for laptops.
- **Workstations doing video editing on disk**: RAM mode is great
  for the OS, but if your project files churn on the same disk,
  there's no benefit vs disk mode.
- **Server admins who need long-term persistent state**: that's the
  exact opposite of our model. Use Debian.
- **Users who don't want to learn ANY Linux**: disk mode is still
  Arch. You'll need at least Arch basics to keep it healthy.

---

## "Which mode do I use?"

| Your use case | Recommended boot mode |
|---|---|
| Daily driver gaming | `gamescope` (default) |
| Retro gaming + media center | `EmulationStation` |
| Single-app TV/kiosk | `kiosk` |
| Programmer workstation | `hyprland` |
| Server / SSH access | `debug TTY` |
| Update the system | `disk (UKI)` |

You can use ALL of them — different jobs, different boots, same machine.
