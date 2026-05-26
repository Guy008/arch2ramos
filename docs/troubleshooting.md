# Troubleshooting

## GRUB boots normal Arch — never reaches the "from RAM" entry

**Symptom:** After running `arch2ram-install`, every reboot lands in your
regular disk-Arch. `journalctl --list-boots` shows no entries with
`BOOT_IMAGE=/vmlinuz-linux ... archisobasedir=...`.

**Root cause:** `GRUB_TIMEOUT` in `/etc/default/grub` is too short
(commonly 0–2 seconds) and `GRUB_DEFAULT` points to the regular entry,
so GRUB auto-boots before you have time to navigate to the RAM entry.

**Fix:**
```bash
sudo sed -i 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=10/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Reboot, then use the arrow keys within 10 seconds to select
"Arch Linux from RAM".

---

## Initramfs drops to emergency shell — "device with UUID ... not found"

**Symptom:** Boot reaches the archiso hook output, then prints something like:
```
ERROR: 'UUID=<...>' device did not show up after 30 seconds
   Falling back to interactive prompt
```

**Root cause:** The `archisosearchuuid` in the GRUB entry doesn't match
the actual UUID of the partition that holds the squashfs. Most often
this happens after re-formatting or moving the partition.

**Fix:**
```bash
# from the emergency shell or after rebooting into disk Arch:
findmnt -n -o UUID /                 # check the real UUID
sudo arch2ram-install                # regenerates the GRUB entry with the current UUID
```

---

## "no root file system image found"

**Symptom:** archiso hook mounts the partition fine but then fails with:
```
ERROR: no root file system image found
```

**Root cause:** `airootfs.sfs` is missing — either `arch2ram-create` was
never run, or the path doesn't match what's in the GRUB entry.

The expected path inside the source partition is:
`<archisobasedir>/<arch>/airootfs.sfs`  →  by default
`/var/lib/arch2ram/x86_64/airootfs.sfs`.

**Fix:**
```bash
sudo arch2ram-create
ls -lh /var/lib/arch2ram/x86_64/   # should show airootfs.sfs
```

---

## After kernel update, RAM boot freezes / panics

**Symptom:** Disk Arch is fine, but the "from RAM" entry hangs partway
through boot or kernel-panics.

**Root cause:** `/lib/modules/$KVER` inside the squashfs doesn't match
the kernel that GRUB is loading. The mkinitcpio pacman hook rebuilds
`initramfs-arch2ram.img` for the new kernel automatically — but the
squashfs is a frozen snapshot and still ships the old `/lib/modules`.

**Fix:** rebuild the squashfs after every kernel update:
```bash
sudo arch2ram-create
reboot
```

---

## copytoram fails with "out of memory" / huge image

**Symptom:** Boot prints `:: Copying rootfs image to RAM...` and then
errors out, or the system OOM-kills processes shortly after boot.

**Root cause:** The squashfs is too large for `copytoram=y` to fit in
tmpfs alongside the cowspace and your applications. Default tmpfs limit
is 75% of RAM (`copytoram_size=75%`).

**Fix options:**
1. Trim the squashfs — edit the `EXCLUDES` list in `arch2ram-create`
   to drop more large directories (Steam libraries, models, datasets, …)
2. Bump tmpfs cap — add `copytoram_size=85%` to the GRUB entry.
3. Disable copytoram — change `copytoram=y` to `copytoram=n`. The squashfs
   then stays on disk and is read on demand. Loses the "zero disk I/O"
   property but works with any image size.

---

## GRUB syntax error after `arch2ram-install`

**Symptom:** `grub-mkconfig` complains about a syntax error near a line
in `grub.cfg`.

**Root cause:** A previous version of `arch2ram-install` removed
`menuentry` lines but left orphaned `linux`/`initrd`/`}` blocks.

**Fix (already in current `arch2ram-install`):** the GRUB entry is
rewritten via a Python parser that splits on `menuentry` boundaries and
replaces the whole block atomically. If you have a hand-broken
`40_custom`, open it and remove the orphaned lines, then re-run
`arch2ram-install`.
