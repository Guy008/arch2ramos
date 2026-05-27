# Batocera ES — סקירה מלאה של כל מה שמופיע בתפריט

נכתב לפני שאנחנו אורזים את ה-fork. **לא כל זה רלוונטי לנו** — צריך לסנן.

מקור: `es-app/src/guis/` (39 קבצי `.cpp`) + `GuiMenu.cpp` (5,533 שורות) + `es_features.cfg` (1,267 שורות PLUS).

---

## 1. תפריטי-על — 10 כניסות ב-MAIN MENU

```
GAME SETTINGS                          ←  הגדרות per-emulator
CONTROLLERS & BLUETOOTH SETTINGS       ←  מקלדות, שלטים, BT
USER INTERFACE SETTINGS                ←  עיצוב, ערכת נושא, פונט
GAME COLLECTION SETTINGS               ←  אוספים, סינון, מועדפים
SOUND SETTINGS                         ←  ראוטינג שמע
NETWORK SETTINGS                       ←  WiFi, hostname, SMB, SSH
SCRAPER                                ←  הורדת מטא-דאטה של משחקים
UPDATES & DOWNLOADS                    ←  ! Content Store + עדכונים
SYSTEM SETTINGS                        ←  ! מתקדמים — סלקטיבי
QUIT                                   ←  סגירה / כיבוי / restart
```

## 2. **UPDATES & DOWNLOADS** — זה האזור שדיברת עליו

```
CONTENT DOWNLOADER          ← GuiBatoceraStore.cpp (547 שורות!)
THEMES                      ← GuiThemeInstaller — חנות ערכות נושא
THE BEZEL PROJECT           ← GuiBezelInstaller — overlays ל-CRT
SOFTWARE UPDATES            ← GuiUpdate — auto-update של המערכת
APPLY UPDATE / START UPDATE
```

### CONTENT DOWNLOADER — איך הוא עובד טכנית
* כל "חבילה" היא pacman-like package שמכיל קוד + meta
* רשימת חבילות נטענת מ-`https://updates.batocera.org/.../store.json`
* כל חבילה: `INSTALL` / `UPDATE` / `REMOVE` — אותו תפריט פר-package
* מה ב-store? סקרייפרים, emulators מתקדמים, themes, bezel-packs, system tools

**שימוש מעשי**: ייתכן שניקח את ה-API ונחבר ל-AUR או pacman שלנו

## 3. **SYSTEM SETTINGS** — האזור המתקדם (זה ה-"קסמים")

```
INFORMATION                 ← cpu, ram, drives, network, gpu
DEVELOPER OPTIONS           ← console logs, debugging
FORMAT A DISK               ← partitioning בתוך ES! (mkfs.ext4, ntfs, ...)
BACKUP USER DATA            ← tar/rsync של userdata
CREATE A SUPPORT FILE       ← דמוי "drag-from-fox" — bug report builder
CLEAR CACHES                ← נקיון תמונות/סקרייפר
SERVICES                    ← samba, ssh, sshd, retroarchivements
HARDWARE                    ← CPU governor, fan curve, brightness, BUTTON LED color
SECURITY                    ← root password, ssh keys
ROOT PASSWORD
ENABLE WIFI / DISABLE PLANE MODE
SCREEN ROTATION             ← למסכים אנכיים
DISPLAY OPTIONS             ← רזולוציה, refresh, scale
STORAGE                     ← STORAGE DEVICE / EXTRA DRIVE FILESYSTEM
EJECT AN EXTRA DISK         ← ejectable USB drives
TIME ZONE / WIFI COUNTRY
DMD                         ← DMD pinball display support!
TOOLS                       ← REDETECT ALL GAMES' LANG/REGION,
                              UPDATE GAMELISTS, RESET CUSTOMIZATIONS,
                              FIND ALL GAMES WITH NETPLAY/ACHIEVEMENTS
```

## 4. **GAME SETTINGS** — Lutris-level per-game

```
SHADERS                     ← CRT/scanlines/HDR/raster
SMOOTH GAMES (BILINEAR)
DECORATIONS                 ← bezels, tattoos, backglass
DECORATION SET / TATTOO CORNER / STRETCH BEZELS
RETROACHIEVEMENTS           ← cheevos integration
RA SETTINGS / RA HARDCORE / RA LEADERBOARDS
NETPLAY SETTINGS            ← multiplayer
BIOS SETTINGS               ← per-emu BIOS check
MISSING BIOS CHECK
AI TRANSLATION              ← תרגום-טקסט-במשחק חי(!)
AUTOCONFIGURE CONTROLLERS
EMULATOR SETTINGS           ← בחירת ליבה (RetroArch core)
PER-GAME OVERRIDES          ← כל זה נשמר פר-משחק
SAVE STATES                 ← display savestate manager
REWIND                      ← אם הליבה תומכת
```

## 5. **USER INTERFACE SETTINGS** — עיצוב מלא

```
THEME SET / THEME OPTIONS / THEME FONT SCALE
GAMELIST VIEW STYLE         ← grid / list / detailed
CAROUSEL TRANSITIONS
SHOW CONTROLLER OVERLAYS
SHOW NETWORK INDICATOR
COMPLETE QUIT MENU
HIDE EMULATIONSTATION WHEN RUNNING A GAME
APPEARANCE / SCREENSAVER SETTINGS
FULL SCREEN MENUS
KID MODE / KIOSK MODE       ← UI restrictions לילדים!
SCREEN READER (TEXT TO SPEECH)  ← אקסיסביליות!
```

## 6. **SOUND SETTINGS**

```
SYSTEM VOLUME / BUTTON LED COLOR
AUDIO OUTPUT                ← HDMI / 3.5mm / USB / Bluetooth
DECREASE FRAME DELAY        ← אוטו for stutter prevention
SONG TITLE DISPLAY DURATION
FAVORITE MUSIC PLAYLIST     ← SAVE/REMOVE/SKIP/SELECT
```

## 7. **CONTROLLERS & BLUETOOTH**

```
PAIRING                     ← GuiBluetoothPair (סריקה)
DEVICES                     ← GuiBluetoothDevices (רשימה + battery)
DEVICE OPTIONS              ← GuiBluetoothDeviceOptions (connect/forget/rename)
WIFI                        ← GuiWifi
KEYBOARD LAYOUT             ← GuiKeyboardLayout
KEYBOARD-TO-PAD MAPPING     ← GuiKeyboardtopads (גיר!)
KEY MAPPING EDITOR          ← GuiKeyMappingEditor
CREATE PADTOKEY PROFILE     ← per-system keybinding overlay
EDIT PADTOKEY PROFILE
HID JOYSTICK DRIVERS
CONTROL ES WITH FIRST JOYSTICK ONLY
SWITCH SOUTH/EAST BUTTONS   ← JPN/PSX-style swap
EMULATED WIIMOTES
GUN MOVE TOLERANCE          ← Sinden Lightgun
```

## 8. **es_features.cfg** — הקסם שמאפשר תפריטים דינמיים

לא קוד. **XML דקלרטיבי**. 1,267 שורות (PLUS) של:

```xml
<emulator name="dolphin" features="videomode, ratio, internal_resolution">
  <feature name="ANISOTROPIC FILTERING" value="anisotropic_filtering">
    <choice name="Off" value="0" /> <choice name="2x" value="1" /> ...
  </feature>
  <feature name="WIDESCREEN HACK" value="widescreen_hack"> ... </feature>
  <feature name="EMULATED WIIMOTES" value="emulated_wiimotes"> ... </feature>
  <!-- ועוד 30+ features ל-dolphin -->
</emulator>
```

ES קורא את זה ב-runtime ובונה את התפריט אוטומטית. **להוסיף "GameMode" כ-feature חדשה = להוסיף 5 שורות XML — אין שינוי קוד**.

---

## 9. מה לא רלוונטי לנו / כדאי לדלג

```
✘ DMD (pinball display)                  — לא רלוונטי לדסקטופ
✘ KODI MEDIA CENTER                       — אנחנו פותרים זה דרך mpv ב-ES
✘ THE BEZEL PROJECT                       — נחמד אבל לא קריטי
✘ STRETCH BEZELS                          — אותה סיבה
✘ EMULATED WIIMOTES                       — emulator-specific
✘ GUN MOVE TOLERENCE                      — Sinden lightgun (לא לנו)
✘ Specific BATOCERA branding              — נחליף ל-arch2ramos
✘ Auto-installer (GuiInstall)             — אנחנו לא installer
```

## 10. מה כן רלוונטי / חשוב מאוד

```
✓ עברית מלאה                              — חובה
✓ BLUETOOTH UI (3 מסכים)                  — חובה (אתה ביקשת)
✓ WIFI / NETWORK UI                       — חובה (אתה ביקשת)
✓ es_features.cfg dynamic menus           — חובה (Lutris-style)
✓ SYSTEM SETTINGS → HARDWARE              — חובה (CPU governor, fan, LED)
✓ FILE BROWSER (F1)                       — אתה ביקשת אתמול
✓ RETROACHIEVEMENTS                       — נחמד מאוד
✓ NETPLAY                                 — נחמד מאוד
✓ SAVE STATES                             — חובה לרטרו
✓ SCRAPER                                 — חובה לרטרו
✓ BIOS check                              — חובה לרטרו
✓ CLEAR CACHES, FIND ALL GAMES            — חובה
✓ Theme installer (downloads from URL)    — נחמד
✓ KID MODE / KIOSK MODE                   — מתאים לרעיון "מערכת מונגשת"
✓ SCREEN READER (TTS)                     — מתאים לרעיון "מערכת מונגשת"
✓ AI TRANSLATION                          — מגניב, מתאים לקלאוד עצמינו!
✓ FORMAT A DISK                           — שימושי לdebug/install
✓ BACKUP USERDATA                         — אפשר לנצל ל-RAM-mode persistence
✓ SERVICES (start/stop)                   — שווה
✓ CONTENT DOWNLOADER (Batocera Store)     — אפשר לחבר ל-AUR!
```

---

## הצעת אסטרטגיה

1. **בכל מקרה לקמפל ולהתקין** את ה-fork — זה כבר נעשה (binary מוכן בשרת)
2. **לוודא תאימות** של ה-`es_systems.cfg` שלנו ל-schema של ה-fork
3. **לזהות** איזה settings הfork קורא מאיפה (כנראה `~/.emulationstation/batocera.conf`) — נצטרך adapter ל-`arch2ram-settings`
4. **להחליף branding** — Batocera לוגו, splash, נוסחאות (`/userdata` → `/home/Guy008/ES`)
5. **לכתוב es_features.cfg משלנו** עבור הknobs של runer.sh (Lutris-style)
6. **לחבר את ה-Content Downloader ל-AUR/yay** (סופר רעיון!)

---

**עכשיו — אני מחכה ל"קסמים" שלך לפני שאני נוגע בPKGBUILD.** מה רצית להראות לי?
