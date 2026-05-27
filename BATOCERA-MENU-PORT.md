# מה לקחת מהתפריט של Batocera

תאריך: 2026-05-27.
מצב: מסמך החלטה לפני המשך עבודה.

---

## 1. התפריט הראשי של Batocera ES (אחרי לחיצה על START)

מ-`GuiMenu.cpp` שלהם — 10 רשומות ברמת העל:

```
┌─ MAIN MENU ──────────────────────────────────────────┐
│  1.  GAME SETTINGS                                    │
│  2.  CONTROLLERS & BLUETOOTH SETTINGS    ← רוצה       │
│  3.  USER INTERFACE SETTINGS                          │
│  4.  GAME COLLECTION SETTINGS                         │
│  5.  SOUND SETTINGS                                   │
│  6.  NETWORK SETTINGS                    ← רוצה       │
│  7.  SCRAPER                                          │
│  8.  UPDATES & DOWNLOADS                              │
│  9.  SYSTEM SETTINGS  (האזור המתקדם)                  │
│ 10.  QUIT                                             │
└──────────────────────────────────────────────────────┘
```

כל אחד מהם מוביל ל-GuiSettings צאצא, ושם יש עוד תפריטים. שכבות
עומק 3-4 בכל ענף.

ספציפית:

### Bluetooth (תוך CONTROLLERS & BLUETOOTH SETTINGS)
מסכים שיש להם:
* `GuiBluetoothDevices` — רשימת המכשירים המשויכים, עם status
  (connected/paired/trusted), עוצמת סוללה, סוג (gamepad/headset/audio)
* `GuiBluetoothPair` — סריקה אינטראקטיבית של מכשירים חדשים, animation
* `GuiBluetoothDeviceOptions` — פעולות פר-מכשיר (connect, disconnect,
  forget, rename, set as default audio)
* גם `GuiControllersSettings` — לראות בקרי-בקר (Xbox/PS/8BitDo)
  ולעשות per-pad mapping

### Network (תוך NETWORK SETTINGS)
* SSID list לבחירה
* WPA password input (מקלדת וירטואלית!)
* hostname / SSID hidden / static IP
* תצוגת IP נוכחי + שמות interfaces
* SSH on/off
* WoL on/off
* Samba toggle ושמות share

### System Settings (התפריט המתקדם)
* Frontend Developer Options
* Multiscreens config
* Format A Disk
* Backup / Restore
* DMD output (מצב פיקסל-ארט במולטיסקרין)
* Services (start/stop של samba, ssh, וכו')

זה הרבה. **וכל זה כתוב ב-C++ בתוך ה-ES fork שלהם.**

---

## 2. מה יש אצלנו עכשיו

ב-`/home/Guy008/ES/settings/` יש 5 .sh shortcuts לכלים חיצוניים:

| .sh | מה רץ | רמת UX |
|---|---|---|
| Bluetooth.sh | `blueman-manager` (GTK) | מקצועי אבל לא של ES |
| WiFi_(NetworkManager).sh | `nm-connection-editor` (GTK) | מקצועי אבל לא של ES |
| Display.sh | `wdisplays` (GTK) | בסיסי |
| Volume.sh | `pavucontrol` (GTK) | מקצועי |
| Files_(Nemo).sh | `nemo` (file manager) | מלא |

הקטגוריה "Bluetooth" שלנו (קטגוריה ראשית) מכילה רק את:
* `Wireless_Controller.sh` — מתחבר ידנית לDS4 ספציפי לפי MAC

מסקנה: **יש לנו את הכלים, אבל ה-UX הוא "פתח אפליקציה GTK חיצונית"**.
זה לא ה-UX של Batocera שהוא מובנה ב-ES.

---

## 3. הפער ושני המסלולים

### מסלול A — לקמפל Batocera ES fork (ה"כפי שהוא מופיע" המלא)

**מה נקבל**:
* את כל ה-GuiBluetooth* ו-Network UI מובנה ב-ES (ניווט עם שלט)
* תרגום עברי (629 מחרוזות מתורגמות מתוך 1245)
* פונט עברי (`opensans_hebrew_condensed`)
* GuiFileBrowser (F1 שאתה תמיד מדבר עליו)
* 40+ מסכי GUI נוספים שלא הזכרנו עדיין
* מקלדת וירטואלית להזנת WiFi password באמצעות שלט

**מה זה דורש**:
* קימפול C++ — `cmake .. && make -j`
* תיקון תלויות שונות מ-Buildroot ל-Arch (boost, libcec, vlc, ...)
* יצירת PKGBUILD לחבילת AUR `arch2ramos-emulationstation`
* כ-2-4 שעות עבודה ראשונית (פעם אחת)
* ה-vanilla emulationstation 2.11.2 הקיים מתפנה

**חסרון**: עבודה גדולה חד-פעמית. בלי זה — אין דרך לקבל UI מובנה ב-ES.

---

### מסלול B — להישאר על vanilla + להרחיב את ה-settings שלנו

**מה נעשה**:
* להוסיף תתי-קטגוריות (folders) ב-`/home/Guy008/ES/settings/`:
  * `Settings/Network/` — entries לכל interface, ל-SSH, ל-WoL, ל-Samba
  * `Settings/Bluetooth/` — Pair New, Disconnect All, List Trusted
  * `Settings/Display/` — Resolution, Multi-Screen, HDR
  * `Settings/Sound/` — Output Device, Volume, Audio Sink
  * `Settings/System/` — Backup, Format Drive, System Update
* כל entry הוא .sh שעוטף כלי קיים בטרמינל או GUI
* navigation: דרך ES carousel (כמו תפריט רגיל)

**יתרון**: עבודה הרבה יותר קלה (יום-יומיים סך-הכל).
**חסרון**: זה לא Batocera UI. זה ES שלנו עם הרבה עוד entries.
**ועוד חסרון**: עברית עדיין לא תעבוד כי vanilla ES לא תומך ב-i18n.

---

## 4. ההמלצה שלי

**מסלול A — לקמפל Batocera ES.**

סיבות:
1. אתה ביקשת ספציפית "כפי שהוא מופיע" — וזה רק קורה ככה.
2. עברית. אתה אמרת בעצמך שאתה רוצה עברית.
3. F1 file browser — דבר שכבר רצית בעבר.
4. ה-UI של Batocera הוא חצי הסיבה שבחרת ב-Batocera בכלל.
5. הכל חופשי — fork קוד פתוח, רישיון MIT.
6. **חבילת AUR אחת** = איך שמישהו אחר יוכל להתקין `arch2ramos`
   בעתיד. כיום אנחנו תלויים בvanilla שלא יבוא איתנו אם נרצה לחלוק.

ברגע שיש לנו את ה-fork מקומפל, אז:
* כל ה-Bluetooth, Network, System UI **מובנה**.
* אנחנו לא צריכים לכתוב `arch2ram-bluetooth-autoconnect` כי
  ה-fork מטפל בזה לבד.
* אנחנו לא צריכים `nm-connection-editor` כי ה-fork פותח את ה-UI שלו.

---

## 5. אם תבחר במסלול A, מה אני עושה

1. **עכשיו**: סוגר את ה-deploy של bluetooth-autoconnect + desktop-mode
   שכבר התחלתי בו (לא חבל לחזור, נשלים אותו ככלי משלים).
2. **אחר כך**: יוצר PKGBUILD לחבילת AUR `arch2ramos-emulationstation`
   על בסיס `/var/tmp/batocera-deep-dive/es-src/`. כולל:
   * pkgver מותאם (2.11.2-batocera-fork)
   * dependencies של Arch (במקום Buildroot)
   * makedepends לcmake/gcc/וכו'
   * הסרת ה-vanilla `emulationstation` הקיים בהתקנה
3. **קימפול** (`makepkg -si`) — 5-15 דקות תלוי במחשב.
4. **התקנה** + הסרת vanilla.
5. **בדיקה** שה-es_systems.cfg שלנו עדיין עובד עם ה-fork (יש
   הבדלי schema קלים — Batocera הוסיפו `<group>`, `<noConfigurator>`,
   וכמה דברים).
6. **תיקונים** במידת הצורך + commit.

זמן צפוי: 2-4 שעות עבודה רציפה.

---

## 6. שאלות

1. **מסלול A** — לקמפל Batocera ES? (ההמלצה שלי)
2. **מסלול B** — להישאר vanilla ולהרחיב settings? (אפשרי אבל פחות מענה לבקשה)
3. **לסיים קודם את ה-deploy הקטן** שכבר התחלתי (bluetooth-autoconnect
   + desktop-mode system) ואחר כך לקפוץ לקימפול?

חכה לי לפני שאמשיך — לא אקמפל בלי אישור.
