# arch2ramos

> **Run your Arch install entirely from RAM — boot in seconds, zero disk I/O after boot, maximum speed.**

The official Arch install ISO already does exactly what this project wants —
it boots a full Arch system from a squashfs into RAM via overlayfs.

`arch2ramos` adapts the same proven mechanism to **your installed Arch
system**: build a squashfs of your `/`, drop in an archiso-style initramfs,
and add a GRUB entry. From then on you can switch between disk-Arch and
RAM-Arch from the GRUB menu.

---

## How it works (the short version)

The pieces that make this work are not invented here — they're the same
pieces Arch's own install medium uses:

- **`mkinitcpio-archiso`** — official Arch package providing the
  `archiso` initcpio hook. The hook can find a squashfs by UUID, copy it
  into a tmpfs, mount it via loop, and set up an overlayfs over it.
- **`mkinitcpio`** — builds our initramfs (`/boot/initramfs-arch2ram.img`)
  against the kernel you already have installed.
- **`mksquashfs`** — packs your running `/` into a compressed read-only image.
- **GRUB** — provides a second menu entry alongside your normal one.

What we add:

1. A small `mkinitcpio` preset that uses the archiso hooks.
2. Two scripts: `arch2ram-install` (one-time setup) and
   `arch2ram-create` (rebuild the squashfs after updates).
3. A GRUB menuentry generator.

---

## Boot flow

```
GRUB → "Arch Linux from RAM"
  ↓
/boot/vmlinuz-linux + /boot/initramfs-arch2ram.img
  ↓
archiso hook in the initramfs:
   ├─ finds the source partition by UUID
   ├─ copies airootfs.sfs into tmpfs   (copytoram=y)
   ├─ loop-mounts the squashfs (ro)
   ├─ mounts a 2 GiB tmpfs as cowspace for writes
   └─ overlayfs: lowerdir=squashfs, upperdir=cowspace → /new_root
  ↓
switch_root /new_root /sbin/init
  ↓
systemd boots your normal Arch desktop, on top of the overlay
```

See `docs/boot-flow.md` for the annotated version with all the params.

---

## Requirements

- **Arch Linux** as the host system (you'll install from the running install).
- **RAM:** enough to fit the squashfs + your runtime workload. A 32 GB
  desktop with a ~5 GB squashfs leaves you ~17 GB free for applications.
- **GRUB** as the bootloader.
- Packages (auto-installed by `arch2ram-install`):
  `mkinitcpio-archiso`, `squashfs-tools`.

---

## Install

```bash
git clone https://github.com/Guy008/arch2ramos.git
cd arch2ramos

sudo scripts/arch2ram-install      # one-time: deps + initramfs + GRUB entry
sudo scripts/arch2ram-create       # build the squashfs of your current system (~5 min)

# bump GRUB timeout if needed so you have time to pick the entry:
sudo sed -i 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=10/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg

reboot
# pick "Arch Linux from RAM" in the GRUB menu
```

---

## What `arch2ram-install` writes

| File                                       | Purpose                                  |
|--------------------------------------------|------------------------------------------|
| `/etc/arch2ram/mkinitcpio.conf`            | hooks + compression for our initramfs    |
| `/etc/mkinitcpio.d/arch2ram.preset`        | preset (picked up by pacman hook)        |
| `/boot/initramfs-arch2ram.img`             | the archiso-style initramfs              |
| `/etc/grub.d/40_custom` entry              | "Arch Linux from RAM" menuentry          |

(Path of the custom GRUB file is auto-detected: `proxifiedScripts/custom`
when grub-customizer is in use.)

## What `arch2ram-create` writes

| File                                          | Purpose                              |
|-----------------------------------------------|--------------------------------------|
| `/var/lib/arch2ram/x86_64/airootfs.sfs`       | the compressed system image          |
| `/var/lib/arch2ram/x86_64/airootfs.sha512`    | checksum (used if `checksum=y` set)  |

The path layout matches the archiso convention
(`<archisobasedir>/<arch>/airootfs.sfs`), which is what the GRUB entry's
`archisobasedir=var/lib/arch2ram` tells the hook to look for.

---

## Update workflow

```bash
# 1. Boot into disk Arch (normal GRUB entry).
sudo pacman -Syu

# 2. Rebuild the squashfs so the RAM image carries the updates.
sudo arch2ram-create

# 3. Reboot → "Arch Linux from RAM".
reboot
```

The initramfs auto-rebuilds via the pacman mkinitcpio hook on kernel
updates — you don't need to re-run `arch2ram-install`. The squashfs is a
frozen snapshot, so it does need a manual `arch2ram-create` after
significant updates (especially kernel updates — the modules in the
squashfs must match the kernel that boots).

---

## Boot params used (and how to override)

The GRUB entry passes:

| Param                              | Value             | Why                                                |
|------------------------------------|-------------------|----------------------------------------------------|
| `archisobasedir=var/lib/arch2ram`  | path prefix       | where on the source partition the squashfs lives   |
| `archisosearchuuid=<ROOT_UUID>`    | filesystem UUID   | partition the hook should mount                    |
| `copytoram=y`                      | force-copy to RAM | otherwise archiso's `auto` mode gives up on >4 GiB |
| `cow_spacesize=2G`                 | overlay capacity  | default 256 MiB fills up fast on a desktop session |

Full param reference: see `/usr/share/doc/mkinitcpio-archiso/README.bootparams`
after installing the package.

---

## Uninstall

```bash
sudo scripts/arch2ram-uninstall            # remove config + GRUB entry
sudo scripts/arch2ram-uninstall --purge    # also delete /var/lib/arch2ram
```

---

## Project layout

```
arch2ramos/
├── README.md                       ← you are here
├── mkinitcpio/
│   ├── arch2ram.conf               → /etc/arch2ram/mkinitcpio.conf
│   └── arch2ram.preset             → /etc/mkinitcpio.d/arch2ram.preset
├── scripts/
│   ├── arch2ram-install            one-time setup
│   ├── arch2ram-create             build/refresh the squashfs
│   └── arch2ram-uninstall          clean removal
├── grub/
│   └── menuentry.example           manual-install template
└── docs/
    ├── boot-flow.md                detailed boot annotated step-by-step
    └── troubleshooting.md          common failure modes
```

---

*Boots in seconds. Runs from RAM. Same mechanism the Arch installer uses,
applied to your own system.*
