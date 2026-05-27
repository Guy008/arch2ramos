# Morning notes — overnight session 2

Date: 2026-05-27 (extended overnight session)
Backup commit (rollback point): `4b9b449` (previous session)
Latest commit: see `git log`

## TL;DR — The project is now a Linux distribution

Last night ended with EmulationStation as a launcher. Tonight we
realized **ES is actually a desktop environment** for the kiosk/console
audience. We extended the architecture accordingly:

```
+ Comprehensive Batocera/Batocera.PLUS source study (sparse clone, read)
+ arch2ram-launch: 30-line dispatch script (vs Batocera's 697-line Python)
+ es_systems.cfg now has 22 categories (was 4):
    Computing:  apps  websites  linux  windows  steam
    Media:      movies  series  livetv  music  photos
    System:     actions  queries (AI stub)
    Retro:      nes snes n64 gb gbc gba nds megadrive dreamcast
                psx ps2 ps3 psp (+ infra for xbox/wii/switch/3ds/etc.)
+ Installed via pacman: retroarch + 9 libretro cores + dolphin-emu + mame + dosbox
+ Major doc rewrite as distribution: README, PHILOSOPHY, AUDIENCES
+ Batocera.PLUS-inspired "websites" category: drop .url file → click → browser
+ "actions" category for grandma-friendly shutdown/reboot/switch-mode
+ "queries" stub for future AI agent integration
```

8+ git commits pushed. **Default boot still gamescope** (untouched).

---

## What ES actually shows now

Boot into **"Arch Linux from RAM (EmulationStation)"** and the carousel
contains (in order):

| Category | Path | Contents | What launching does |
|---|---|---|---|
| **Apps** | `~/ES/apps` | 111 .desktop files (filtered visible) | `gtk-launch <name>` via runer.sh |
| **Websites** | `~/ES/websites` | 9 .url files (YouTube, Mako, Ynet, Gmail, WhatsApp, Wikipedia, Maps) | Chrome `--start-fullscreen --app=<url>` |
| **Linux Games** | `~/ES/linux` | 12 game wrappers (auto-discovered from `/media/G/Linux/`) | runer.sh `<wrapper>` |
| **Windows Games** | `~/ES/windows` | 20 game wrappers (via wine, biggest-exe heuristic) | runer.sh `wine <exe>` |
| **Steam** | `~/ES/steam` | Steam Big Picture launcher | `steam -bigpicture` |
| **Movies** | `/media/B/Movies` | mkv/mp4/avi files | mpv fullscreen with hwdec |
| **Series** | `/media/A/Series` | mkv/mp4/avi files | mpv fullscreen |
| **Live TV** | `~/ES/livetv` | 6 HLS channel wrappers (Channel 11/12/13/14/Sport5 + IPTV playlist) | mpv with cache |
| **Music** | `~/ES/music` | empty (drop .mp3/.flac) | mpv `--no-video` |
| **Photos** | `~/ES/photos` | empty (drop .jpg/.png) | feh slideshow |
| **System Actions** | `~/ES/actions` | 8 system tasks (shutdown, reboot, mode switch, status) | bash directly |
| **AI Queries** | `~/ES/queries` | 3 sample .prompt files | konsole + claude-cli or ollama (or stub) |
| **NES** | `/media/R/batocera/roms/nes` | 16 NES ROMs | retroarch + mesen core |
| **SNES** | `/media/R/batocera/roms/snes` | 884 SNES ROMs | retroarch + bsnes core |
| **N64** | `/media/R/batocera/roms/n64` | 299 N64 ROMs | retroarch + mupen64plus-next |
| **GB / GBC / GBA / NDS** | corresponding `/media/R/...` paths | varies | retroarch + gambatte/mgba/melonds |
| **Genesis** | `/media/R/batocera/roms/megadrive` | varies | retroarch + genesis-plus-gx |
| **Dreamcast** | `/media/R/batocera/roms/dreamcast` | varies | retroarch + flycast |
| **PSX** | `/media/R/batocera/roms/psx` | 6 ROMs | retroarch + beetle-psx-hw |
| **PS2** | `/media/R/batocera/roms/ps2` | 34 ROMs | pcsx2 (if installed — currently missing from repos) |

That's a working multi-purpose computing environment. No window manager,
no desktop. **The launcher IS the environment.**

---

## What was installed via pacman (no AUR, no risky builds)

```
retroarch                        emulator front-end
libretro-mesen                   NES (accurate)
libretro-bsnes                   SNES (accurate)
libretro-mupen64plus-next        N64
libretro-genesis-plus-gx         Genesis/MD/MS/SG-1000
libretro-gambatte                GB/GBC
libretro-mgba                    GBA
libretro-melonds                 NDS
libretro-beetle-psx-hw           PSX (HW accelerated)
libretro-flycast                 Dreamcast
libretro-core-info               core metadata
libretro-dolphin                 GameCube/Wii (libretro variant)
dolphin-emu                      GameCube/Wii (standalone)
mame                             MAME standalone
dosbox                           DOS games
```

NOT installed (need AUR or unavailable):
```
pcsx2                  PS2 — not in repos right now
ppsspp                 PSP — install failed, retry tonight
rpcs3                  PS3 — large AUR build
ryujinx                Switch — Mono dep, AUR
cemu                   Wii U — AUR
xemu / xenia           Xbox / Xbox 360 — AUR
azahar / citra-qt      3DS — AUR
dosbox-staging         DOS — AUR
duckstation            PSX standalone — AUR (libretro beetle is fine)
```

**To add later**: `yay -S ppsspp pcsx2-git rpcs3 ryujinx-bin cemu azahar-bin`

---

## What to test in the morning

### 1. Reboot to "Arch Linux from RAM (EmulationStation)"
Should see ALL the categories above. Try:
- **Websites → YouTube** (Chrome opens fullscreen on youtube.com)
- **Apps → Chrome** (just normal Chrome, runs via gtk-launch)
- **Linux Games → Cuphead** (or similar)
- **Live TV → Channel 11** (if your HLS server is up, mpv plays)
- **System Actions → Switch to Hyprland** (reboots into hyprland mode)
- **NES → any ROM** (retroarch+mesen opens fullscreen)
- **System Actions → System Status** (opens konsole with diagnostics)

### 2. Verify the modes still work
- gamescope (default) — should be identical to last test
- hyprland — should be identical
- disk-mode — should work for updates

### 3. AI queries (stub)
Click an AI Query entry. If you have `claude` (Claude Code CLI) or
`ollama` installed and configured, it'll actually run the prompt.
Otherwise it shows a placeholder asking you to install one. **Wiring
this up to a real API is a separate session.**

---

## Open decisions for you

1. **Default boot mode** — keep gamescope, or switch default to ES?
   ```bash
   # to make ES default:
   sudo arch2ram-install --default=es
   ```

2. **Strip-packages** — activate the 3 GB image trim?
   ```bash
   sudo cp /etc/arch2ram/strip-packages.conf.example /etc/arch2ram/strip-packages.conf
   # edit to taste, then:
   sudo arch2ram-update
   ```

3. **AI integration** — wire `queries` category to actual API?
   - Easy: `claude` (Claude Code CLI) — works locally
   - Easy: `ollama` — local LLMs
   - Need API key: OpenAI / Anthropic Cloud / Gemini

4. **Custom ES build with hotkeys** — Batocera ES has F1=file-browser
   etc.; we'd need to compile our own to add custom keybindings. This
   is the next big feature. Estimate: 2-3 hours focused work.

5. **Movies/Series scraping** — ES can scrape metadata from
   screenscraper.fr / thegamesdb.net. Free with API signup. Would give
   you box art, descriptions, release dates for movies/series too.

---

## Files added/modified overnight

```
arch2ramos/
├── README.md                          ← rewritten as distro pitch
├── PHILOSOPHY.md                      ← NEW: design principles
├── AUDIENCES.md                       ← NEW: 7 target user types
├── MORNING-NOTES.md                   ← this file (updated)
├── scripts/
│   ├── arch2ram-launch                ← NEW: 30-line dispatcher
│   ├── arch2ram-es-discover           (last session)
│   ├── arch2ram-emulationstation      (last session, untouched)
│   ├── arch2ram-create                ← updated for strip-packages
│   └── arch2ram-install               ← updated for ES install + run discover
├── emulationstation/
│   └── es_systems.cfg                 ← 22 systems (was 4)
└── strip-packages.conf.example        (last session)
```

```
/home/Guy008/ES/                       ← user content (on disk, not in squashfs)
├── apps/                              111 .desktop symlinks
├── websites/                          9 .url shortcuts
├── linux/                             12 game wrappers
├── windows/                           20 game wrappers
├── steam/                             1 BPM entry
├── movies/  series/                   (point to /media/B/Movies etc.)
├── livetv/                            6 HLS wrappers
├── music/  photos/                    (empty — drop your files)
├── actions/                           8 system actions
└── queries/                           3 sample AI prompts
```

```
/usr/local/bin/                        ← installed tonight
├── arch2ram-launch                    central dispatcher
├── arch2ram-emulationstation          ES boot mode launcher
├── arch2ram-es-discover               populate ~/ES from real sources
├── arch2ram-update                    refresh cycle (disk mode)
├── arch2ram-create                    build squashfs
└── arch2ram-drm-fixup                 simpledrm unbind (boot)

/etc/emulationstation/                 ← installed tonight (in squashfs)
├── es_systems.cfg                     systems definition
└── themes/carbon/                     Carbon theme + symlinks for categories
```

---

## The big realization (philosophy delta from last night)

**Last night**: ES is a launcher for games + a few apps.

**Tonight**: ES is a **desktop environment**. Every "category" is just a
shell-command dispatcher. The launch chain is:
```
ES menu pick
  → arch2ram-launch SYSTEM ROM
    → case SYSTEM: pick the runner + args
      → runer.sh wraps for GPU/CPU acceleration
        → exec target
```

This means literally **any shell-runnable thing** can be a category.
Tonight's additions prove the pattern:
- Websites = browser launching URL from text file
- Live TV = mpv launching HLS stream
- System Actions = bash running grub-reboot
- AI Queries = (stub) curl to AI API

Future categories that fit the pattern (deferred):
- SSH connections (drop .host file → ssh to it)
- Network shares (drop .share → mount and open file manager in it)
- Snippets (drop .txt with shell command → execute it)
- Voice notes (drop .ogg → transcribe via whisper → save → open as text)

**This is what makes arch2ramos a distribution, not a tool.** The
infrastructure (RAM boot + setpriv + drm-fixup + multi-mode GRUB +
runer.sh + arch2ram-launch + ES) is the **base of an OS** that
**non-technical users can use** without knowing Linux exists.

---

## Things deliberately NOT done (per instructions / risk)

- ❌ No reboot during the session
- ❌ No modifications to linux-guy (memory: hands off)
- ❌ No deployment to headless server (memory: hands off)
- ❌ No custom ES compile (too risky autonomously — needs a session)
- ❌ No Batocera image download/extract (low ROI vs sparse git clone)
- ❌ No AUR builds (risky, slow, may break dependencies)
- ❌ No overlay-persistence experiments (touches boot mechanism)

---

Sleep well. Walk through the menus in the morning. Tell me which
categories are gold and which need pruning. We'll iterate.
