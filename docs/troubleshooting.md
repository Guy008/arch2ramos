# Troubleshooting

## Boot appears to go straight to normal Arch (RAM entry never runs)

**Symptom:** After installing, rebooting always boots normal Arch. `journalctl --list-boots`
shows no entry with `BOOT_IMAGE=/arch2ram-vmlinuz` — only the normal kernel.

**Root cause:** `GRUB_TIMEOUT` is too short (e.g. `"2"` seconds) and `GRUB_DEFAULT="Arch Linux"`
auto-selects the normal entry before you can navigate to "Arch Linux from RAM".

**Fix:**
```bash
# Increase timeout so you have time to select the RAM entry
sudo sed -i 's/GRUB_TIMEOUT=.*/GRUB_TIMEOUT="10"/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Then reboot and use arrow keys within 10 seconds to select "Arch Linux from RAM".

---

## Black screen after selecting RAM entry (no text output)

**Symptom:** GRUB loads the entry, screen goes black, nothing visible.

**Root cause:** `quiet` suppresses kernel output; nvidia framebuffer not active without its driver.

**Fix:** Use `nomodeset` and remove `quiet` from the GRUB entry:
```
linux  /arch2ram-vmlinuz console=tty0 console=tty1 rdinit=/init nomodeset
```
`nomodeset` forces VGA text mode — you will see all init script output.

---

## losetup: No such device or address (ENXIO)

**Symptom:** `[3/5] Mounting squashfs...` fails with "No such device or address".

**Root cause:** Kernel module version mismatch. The running kernel (e.g. `linux`) doesn't match
the modules bundled in the initrd (built for `linux-zen`). Check:
```bash
uname -r                   # running kernel
ls /boot/arch2ram-vmlinuz  # which kernel was copied
```

**Fix:** Re-run `arch2ram-install`. The `copy_kernel()` function explicitly prefers `linux-zen`.
If both kernels are installed, make sure the zen kernel is running when you install.

---

## losetup: No such file or directory (ENOENT on /dev/loop0)

**Symptom:** `losetup` fails because `/dev/loop0` doesn't exist.

**Root cause:** Kernel compiled with `CONFIG_BLK_DEV_LOOP_MIN_COUNT=0` — no static loop nodes.
Must allocate via `/dev/loop-control`.

**Fix (already in init):** The init script uses `losetup -f` via loop-control, then `mknod` if
the node is missing:
```sh
mknod /dev/loop-control c 10 237 2>/dev/null || true
LOOP_DEV=$(losetup -f 2>/dev/null)
[ ! -e "${LOOP_DEV}" ] && mknod "${LOOP_DEV}" b 7 "${LOOP_DEV##*loop}"
```

---

## GRUB syntax error / grub-mkconfig fails

**Symptom:** `grub-mkconfig` reports "syntax error" around a specific line in grub.cfg.

**Root cause:** Old `add_grub_entry()` used `grep -v LABEL` which stripped `menuentry` lines
but left orphaned `insmod / linux / initrd / }` blocks from previous entries.

**Fix (already in arch2ram-install):** The function now uses python3 to parse and replace
entire menuentry blocks cleanly.

---

## /etc/grub.d/40_custom not found

**Symptom:** `arch2ram-install` writes the GRUB entry but it never appears in grub.cfg.

**Root cause:** System uses grub-customizer, which uses `proxifiedScripts/custom` instead
of `40_custom`.

**Fix (already in arch2ram-install):** Script auto-detects the correct path:
```bash
if [[ -f "/etc/grub.d/proxifiedScripts/custom" ]]; then
    GRUB_CUSTOM="/etc/grub.d/proxifiedScripts/custom"
else
    GRUB_CUSTOM="/etc/grub.d/40_custom"
fi
```
