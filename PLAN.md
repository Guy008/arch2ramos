# arch2ramos — תוכנית עבודה לאחר ההבנה ש"אין WM, אין DE"

תאריך טיוטה: 2026-05-27. שלב: **טיוטה ראשונה לדיון.**

---

## 0. החזון — מה בעצם בונים

> **A bulletproof Linux distribution where the launcher IS the desktop.
> No window manager, no desktop environment, no taskbar — just a single
> menu that runs any binary the system can exec. Everything reachable
> with one key. The user never sees a file manager or settings dialog
> by default.**

זה לא Batocera-with-Arch. זה לא SteamOS. זה משהו אחר —
ה-**launcher-as-OS** philosophy שהבנו הלילה.

מה זה אומר טכנית?
- **אין compositor אישי** בכל מצב — gamescope (או cage) הוא דק, רק תופס DRM.
- **המסך תמיד מוקדש לאפליקציה אחת** — אין חלונות, אין tray, אין taskbar. אלא אם כן עוברים למצב שולחן עבודה 
- **כל ה-system controls (volume / brightness / hotkeys) חיים ב-kernel/evdev layer**, לא בDE.
- **מעבר בין אפליקציות = launcher** (ES) צץ מעל הריצה הנוכחית, או בסיום.

---

## 1. המצב הנוכחי — מה כבר עובד

עברנו את זה כבר בלילות הקודמים. סטטוס:

| שכבה | מצב | הערות |
|---|---|---|
| Boot from RAM | ✅ | squashfs + overlayfs, copytoram=y |
| 5 GRUB modes | ✅ | gamescope/ES/cage/hyprland/debug | --- כאן צריך להחליף ל1 כאשר המעברים יהיו דרך מערכת ההפעלה עצמה 
| arch2ram-launch dispatcher | ✅ | 27+ system case-statement |
| 25+ ES categories | ✅ | apps/websites/music/.../bluetooth |
| ~2,330 entries auto-discovered | ✅ | discovery scripts |
| Hebrew tags + AI + bookmarks | ✅ | infrastructure layer |
| GRUB → mode switching | ✅ | systemctl Conflicts= between launchers |
| RAM-mode hotkey wizard | ❌ | חסר es_input.cfg טוב + hotkeys global |
| Mouse hide / show | ❌ | חסר unclutter daemon + binding |
| Auto-driver detection | ❌ | יש drm-fixup אבל ידני |
| Multi-screen aware | ❌ | hyprland-only |
| Kiosk lockdown (no Ctrl+Alt+Fx) | ❌ | חסר S33disablealtfn equivalent |
| layered squashfs | ❌ | יחיד 4.5GB |

---

## 2. החזון — ארכיטקטורה מלאה

```
┌─────────────────────────────────────────────────────────────────┐
│  KERNEL LAYER                                                    │
│   ├─ evdev (raw input)                                          │
│   ├─ DRM (display)                                              │
│   └─ overlayfs (root)                                           │
└─────────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────────┐
│  KERNEL-LEVEL DAEMONS (no DE dependency)                         │
│   ├─ triggerhappy (kernel hotkeys → arbitrary commands)         │
│   ├─ udev (device events → udev rules)                          │
│   └─ acpid (power button / lid)                                 │
└─────────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────────┐
│  SYSTEM SERVICES (arch2ram-*)                                    │
│   ├─ arch2ram-drm-fixup       (simpledrm unbind)                │
│   ├─ arch2ram-pick-primary-gpu  (NEW — port S30checkprime)      │
│   ├─ arch2ram-fans            (NEW — port S70fans pattern)      │
│   ├─ arch2ram-mouse           (NEW — port mouse-pointer)        │
│   ├─ arch2ram-settings        (NEW — port batocera-settings)    │
│   ├─ arch2ram-boot-mode       ✅ (already built)                │
│   └─ arch2ram-disablealtfn    (NEW — port S33)                  │
└─────────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────────┐
│  COMPOSITOR (one of these, picked by mode)                       │
│   ├─ gamescope         (default, gaming + steam BPM)            │
│   ├─ cage              (single-app strict, internet-cafe)       │
│   ├─ ES (gamescope)    (launcher mode)                          │
│   └─ hyprland          (escape hatch — for power user mode)     │
└─────────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────────┐
│  USER-FACING LAUNCHER                                            │
│   EmulationStation 2.11.2 (or our forked build later)           │
│   25+ categories — auto-discovered + auto-generated             │
└─────────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────────┐
│  RUNTIME WRAPPER                                                 │
│   arch2ram-launch SYSTEM ROM                                     │
│   → runer.sh wraps for GPU/Vulkan/MangoHud accel                │
│   → exec target binary                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. תוכנית עבודה — 4 sprints

### Sprint 1 (שעה — מאמץ נמוך, אימפקט גבוה) — "Universal Hotkeys"

מטרה: hotkeys שעובדים **בכל boot mode** (gamescope / ES / cage / hyprland / TTY).

#### S1.1 — Install + configure triggerhappy
```bash
sudo pacman -S triggerhappy xosd xdotool unclutter yad
sudo systemctl enable --now triggerhappy
```

#### S1.2 — Hotkey config file
`/etc/triggerhappy/triggers.d/arch2ram.conf`:
```
# arch2ramos universal hotkeys (kernel-level via evdev)
KEY_F1+KEY_LEFTMETA   1  /usr/local/bin/arch2ram-launch shares /home/Guy008/ES/drives/Home.sh
KEY_F2+KEY_LEFTMETA   1  /usr/local/bin/arch2ram-help
KEY_F3+KEY_LEFTMETA   1  /usr/local/bin/arch2ram-mouse invert
KEY_F4+KEY_LEFTMETA   1  konsole
KEY_F12+KEY_LEFTMETA  1  /usr/local/bin/arch2ram-screenshot
KEY_SYSRQ             1  /usr/local/bin/arch2ram-screenshot
KEY_LEFT+KEY_LEFTMETA   1  amixer set Master 2%-
KEY_RIGHT+KEY_LEFTMETA  1  amixer set Master 2%+
KEY_DOWN+KEY_LEFTMETA   1  amixer sset Master toggle
```

#### S1.3 — `arch2ram-mouse` script (port of PLUS's mouse-pointer)
- State via `/tmp/arch2ram-mouse-{on,off}` flag files
- `unclutter-remote` for hide/show
- Modes: `on`, `off`, `invert`

#### S1.4 — `arch2ram-screenshot` (port of batocera-print-screen)
- `import -window root -display :0.0 /home/Guy008/Screenshots/$(date).png`
- `osd_cat` notification

#### S1.5 — `arch2ram-help` (port of batocera-hotkey-help)
- konsole + cat << EOF showing ALL hotkeys
- F2 alone (no Win) → quick OSD "Press Win+F2 for help"

**Outputs**: 5 scripts + 1 conf file. ~200 lines total. **Testable independently of mode.**

---

### Sprint 2 (שעה — לוקח את ה-launcher לרמת ייצור)

מטרה: ES + מעטפת startup חזקה כמו Batocera's emulationstation-standalone.

#### S2.1 — arch2ram-emulationstation wrapper rewrite
Current: ~30 lines, just exec gamescope+ES.
Target: 100+ lines, with:
- `/tmp/shutdown.please` / `reboot.please` / `restart.please` triggers
- ES exit loop (if ES exits, check trigger files, restart unless shutdown)
- Pre-launch: wait for compositor, apply locale, load es_input.cfg
- Post-exit: clean up, handle triggers

#### S2.2 — es_input.cfg expanded
- Keyboard mapping (already done — but expand with hotkey button)
- DualShock 4 / 5 mapping  
- Xbox 360 / One / Series controller mapping
- 8BitDo Pro 2 mapping
- Generic HID fallback

Take ALL of Batocera's input config — they have 80+ controllers mapped. Just copy.

#### S2.3 — arch2ram-disablealtfn
Port S33: `dumpkeys | grep -vE "alt keycode.*Console_" | loadkeys`.
Disable Ctrl+Alt+F1..F12 in all RAM modes.

---

### Sprint 3 (שעה — system services)

מטרה: כל ה-"plumbing" שכתוצאה ממנו המערכת **עולה מהמפעל מהר ובחוזקה**.

#### S3.1 — arch2ram-pick-primary-gpu.service
Port S30checkprime — אם יש >1 GPU, אתר את זה שבאמת מחובר למסך, mark as primary.
**מחליף את ה-drm-fixup הידני.**

#### S3.2 — arch2ram-settings (the C-or-shell tool)
- Single config: `/etc/arch2ram/settings.conf`
- `arch2ram-settings get <key>` / `set <key> <value>`
- שאר הסקריפטים קוראים דרכו, לא קבצים מפוזרים

(אופציה: ELF binary לביצועים, אם מסתבר שזה מובחן בboot time.)

#### S3.3 — Init ordering refactor
שכתוב של ה-services שלנו כך שיש להן סדר הגיוני (לא דורש שינוי מהיר, אבל documentation):
```
S05  arch2ram-drm-fixup       (early — DRM ready)
S05  arch2ram-pick-primary-gpu (parallel — GPU choice)
S30  arch2ram-disablealtfn    (after udev settled)
S31  arch2ram-boot-mode       (start launcher service)
S70  arch2ram-fans            (late — sensors must be loaded)
```

---

### Sprint 4 (שעה — visual polish + GRUB cleanup)

מטרה: clean shipping. אחת לכמה ימים אנחנו אמורים לרצות לראות שיפור ויזואלי.

#### S4.1 — GRUB cleanup to 2 entries
- **Arch Linux from RAM** (no mode param → boot-mode picks default = ES)
- **Arch Linux (disk, UKI)** (for updates)
- All mode switching done IN-SESSION via ES Actions or hotkey

#### S4.2 — Custom theme polish
- Replace symlinks (apps → pc) with proper SVGs (we did this — but check quality)
- Add Hebrew BiDi rendering for ES system names

#### S4.3 — Final squashfs + commits + docs
- All changes baked
- README + PHILOSOPHY + AUDIENCES updated
- Tag release v0.2

---

## 4. החלטות שצריך שתבחר בהן עכשיו (לפני ביצוע)

### D1: סדר הביצוע
- [ ] Sprint 1 → 2 → 3 → 4 (כפי שהצעתי)
- [ ] Sprint 4 (visual) קודם (הכי בולט)
- [ ] Sprint 2 (launcher) קודם (הכי משפיע על UX)
- [ ] Sprint 1+3 במקביל (hotkeys + services הם independent)

### D2: עד כמה אגרסיבי לפורט מ-Batocera
- [ ] **Minimal**: רק 6 הירוקות (hotkeys, mouse, kiosk lock, OSD, screenshot, help)
- [ ] **Medium**: כולל S30checkprime + arch2ram-settings infrastructure
- [ ] **Maximal**: גם layered squashfs + S70fans pattern + per-device quirks library

### D3: ES build
- [ ] **דהוי vanilla** ES 2.11.2 — אבל לאמץ Batocera's es_input.cfg
- [ ] **קומפילציה של Batocera's ES fork** (3-4 שעות עבודה, אבל מקבלים F1 file browser + 40+ GUI screens)
- [ ] **אפשרות C**: fork קל משלנו עם 2-3 patches (F1 + hotkey)

### D4: GRUB
- [ ] לקצץ ל-2 entries (כפי שהצעת)
- [ ] להשאיר 5 entries (גמישות)
- [ ] לקצץ ל-3 (RAM + disk + debug TTY recovery)

### D5: היכן אתה רוצה לעבוד מ-(disk vs RAM)?
- [ ] disk mode (יציב, אפשר reboot ולבדוק)
- [ ] RAM mode עכשיו (כל השינויים בoverlay, אם משהו נשבר אז reboot ונקי)

---

## 5. סיכון ניהול

| סיכון | סבירות | חומרה | mitigation |
|---|---|---|---|
| triggerhappy לא יעבוד טוב בRAM mode | נמוכה | בינונית | בדיקה מקומית קודם (disk mode) |
| es_input.cfg של Batocera לא יתאים | בינונית | נמוכה | יש fallback של keyboard mapping בסיסי |
| arch2ram-pick-primary-gpu ישבר משהו | בינונית | גבוהה | מוגן ע"י drm-fixup כ-safety net |
| disablealtfn יעצור אותי ב-TTY | גבוהה | בינונית | רק במצבים RAM, לא disk |
| Layered squashfs לא יעלה | גבוהה | גבוהה | רק ב-Sprint maximal, opt-in |

---

## 6. הצלחה — איך נדע שזה הצליח

אחרי 4 sprint-ים (4 שעות עבודה):

```bash
# Sanity:
arch2ram-help                   → מראה כל hotkeys ב-OSD/konsole
Win+F3                          → עכבר נעלם
Win+F4                          → konsole
Win+F12                         → screenshot + osd "saved"
F2 alone                        → osd_cat hint "press Win+F2"
arch2ram-settings get version   → 0.2
```

**אם הכל עובד = הצלחה.** אם משהו אחד שובר — אנחנו יודעים בדיוק מה.

---

## 7. שאלות אליך לפני שאני יוצא לדרך

1. **המוטיבציה כללית** — האם זה בכיוון? צריך לשנות תכלית?
2. **D1-D5** — מה בחרת?
3. **מה חסר בתוכנית** שכן רצוי?
4. **מה יש בתוכנית** שאתה לא רוצה?

---

**זה הטיוטה. תקרא, תעיר. נדבר.**
