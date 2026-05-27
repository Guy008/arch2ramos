# Morning notes — overnight work summary

Date: 2026-05-27 (overnight session)
Backup commit (rollback point): `dba4f2e`
Final commit: `e974d1a` (+ pending install.sh update)

## TL;DR

```
+ Phase 1: package-level stripping via pacman -Qlq (opt-in via /etc/arch2ram/strip-packages.conf)
+ Phase 2: runer.sh — 3 bug fixes (backup at runer.sh.bak.YYYYMMDD-HHMMSS)
+ Phase 3: EmulationStation 2.11.2 installed from AUR (~149MB)
+ Phase 4: /etc/emulationstation/es_systems.cfg with REAL category sources:
            apps    ← /usr/share/applications/*.desktop (111 visible apps)
            linux   ← /media/G/Linux/ (12 games auto-discovered)
            windows ← /media/G/Windows/ (20 games via wine)
            steam   ← Steam Big Picture single entry
+ Phase 5: arch2ram-es-discover script — populates ~/ES/ from real sources
+ Phase 6: arch2ram.mode=emulationstation — new boot mode, new GRUB entry
+ Phase 7: tested ES in current X11/Wayland session — loads OK, finds all 4
+ Phase 8: new squashfs built (4.5GB)
+ Phase 9: arch2ram-install updated to handle ES end-to-end + auto-discovery
```

6 git commits pushed. **Default boot is still gamescope** (untouched). ES is GRUB index 1.

## Lutris integration is automatic

Lutris exports games as `.desktop` files into `/usr/share/applications/`
(via "Create application menu shortcut" per-game). These will appear
in the **Apps** category automatically — including all per-game settings
(HDR, FSR, runtime overrides, etc.) baked into Lutris's launch command.

To add a Lutris game to ES:
1. In Lutris: right-click game → "Create application menu shortcut"
2. Run: `arch2ram-es-discover` (or it'll be picked up next time install runs)
3. Game appears in Apps in ES.

Same flow for Steam — "Create desktop shortcut" from Steam right-click.

---

## GRUB now has 7 entries

```
1. Arch Linux from RAM (gamescope)         ← default
2. Arch Linux from RAM (EmulationStation)  ← NEW — test this in the morning
3. Arch Linux from RAM (kiosk)
4. Arch Linux from RAM (hyprland)
5. Arch Linux from RAM (debug TTY)
6. Arch Linux (disk, UKI — for updates)
7. Arch Linux (disk, linux-zen kernel)
```

---

## What to test in the morning

### 1. EmulationStation mode (the main event)

Reboot, pick **"Arch Linux from RAM (EmulationStation)"**.

**Expected**:
- gamescope at 4K@60 takes over
- ES boots, shows a system carousel with 3 systems: **Games / Apps / Tools**
- Each system has at least 1 entry:
  - Games: `steam.sh`
  - Apps: `chrome.sh`, `firefox.sh`, `mpv.sh`, `dolphin.sh`
  - Tools: `konsole.sh`, `htop.sh`
- Pick one → exec via `runer.sh` (mangohud + RADV env vars) → app launches fullscreen
- Exit app → return to ES

**Likely first-time issues**:
- ES first launch may show "Configure Input" wizard. Press any key on keyboard, follow the prompts. Default bindings: arrows = D-pad, Enter = A, Esc = B.
- Theme warnings in `/home/Guy008/.emulationstation/es_log.txt` are cosmetic (symlinked carbon themes missing some icon SVGs)
- If ES doesn't find the systems: verify `/home/Guy008/ES/{games,apps,tools}/` exists and has `.sh` files

### 2. Verify gamescope mode still works (not broken)

Reboot, pick "Arch Linux from RAM (gamescope)". Should be identical to last night.

### 3. Update workflow

Reboot to disk (`Arch Linux (disk, UKI)`), then `sudo arch2ram-update` to refresh squashfs cleanly.

---

## Decisions waiting for you

### 1. Activate package stripping?

`/etc/arch2ram/strip-packages.conf.example` exists with my draft list:
- All 3 DMs (gdm/sddm/lightdm) → ~150 MB
- Plasma + GNOME + Cinnamon → ~2.7 GB
- yay/paru/build-tools/kernel-headers → ~600 MB
- **Estimated savings: ~3.5 GB** before zstd compression → ~1.3 GB on squashfs

To activate:
```bash
sudo cp /etc/arch2ram/strip-packages.conf.example /etc/arch2ram/strip-packages.conf
# Edit to your preference — comment out anything you DO want in RAM
sudo arch2ram-update   # from disk mode
```

**Don't activate unless you've reviewed the list** — some things may be deps of stuff you keep.

### 2. ES becomes default?

Right now gamescope is default. If you want ES as default:
```bash
sudo arch2ram-install --default=es --no-hyprland   # or with whichever flags
# This re-emits GRUB with ES first → becomes index 0
```

Or just edit `/boot/grub/grub.cfg` directly: change `set default="0"` (currently means gamescope) to `set default="1"` (which is now ES).

### 3. runer.sh fixes — keep or revert?

Backup at `/home/Guy008/Scripts/runer.sh.bak.20260527-025029`. Diff is small:
1. Added `config_gamemode_run` call in `main()` (was defined but never called)
2. Fixed `launch_application` to read `$1` as APP (was always empty string)
3. Removed `VK_ICD_FILENAMES`/`VK_DRIVER_FILES` for amd_pro_icd (conflicted with `AMD_VULKAN_ICD=RADV`)

If you preferred the original behavior:
```bash
mv /home/Guy008/Scripts/runer.sh.bak.20260527-025029 /home/Guy008/Scripts/runer.sh
```

### 4. ES theme

Using Carbon (the classic RetroPie theme). For TV viewing at 4K, the icons are small. Alternatives to consider:
- **es-theme-pixel** — minimalist, large fonts
- **es-theme-recalbox** — bigger, more modern (used by Recalbox)
- **Custom theme** — I could draft one with Games/Apps/Tools-specific icons

---

## What I deliberately did NOT do

- ❌ Did not reboot the machine (would have lost the session)
- ❌ Did not enable strip-packages.conf (your review needed)
- ❌ Did not make ES the default boot (preserving last night's working state)
- ❌ Did not modify linux-guy (per memory)
- ❌ Did not propose anything for the headless server (per memory)
- ❌ Did not remove anything from disk (only excluded from squashfs)
- ❌ Did not push to git as "force" — all linear commits with full messages

---

## Files added overnight

```
arch2ramos/
├── strip-packages.conf.example                ← Phase 1
├── scripts/
│   ├── arch2ram-create                        ← updated for strip-packages
│   ├── arch2ram-emulationstation              ← Phase 5
│   ├── arch2ram-gamescope                     ← (committed last session)
│   ├── arch2ram-install                       ← updated for ES mode
│   └── arch2ram-update                        ← (committed last session)
├── systemd/
│   └── arch2ram-emulationstation.service      ← Phase 5
├── emulationstation/                          ← Phase 4 (new dir)
│   ├── README.md
│   └── es_systems.cfg
└── MORNING-NOTES.md                           ← this file
```

```
/home/Guy008/Scripts/
├── runer.sh                                   ← 3 bug fixes
└── runer.sh.bak.20260527-025029               ← original backup
```

```
/home/Guy008/ES/
├── games/steam.sh
├── apps/chrome.sh apps/firefox.sh apps/mpv.sh apps/dolphin.sh
└── tools/konsole.sh tools/htop.sh
```

```
/home/Guy008/.emulationstation/
├── es_systems.cfg
└── themes/carbon/  (RetroPie carbon theme, full)
```

---

## Quick sanity checks if something looks wrong

```bash
# Did the squashfs include ES?
sudo mkdir -p /mnt/check
sudo mount /dev/$(blkid -U a92e9313-9f98-4c10-90e0-f63e30688b32 | xargs basename) /mnt/check
sudo unsquashfs -ll /mnt/check/var/lib/arch2ram/x86_64/airootfs.sfs | grep -c emulationstation
# Should be > 0

# What's the squashfs size + timestamp?
ls -la /mnt/check/var/lib/arch2ram/x86_64/airootfs.sfs
# Last build: 02:59

sudo umount /mnt/check
```

---

Sweet dreams. Talk in the morning.
