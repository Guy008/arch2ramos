# arch2ramos

Boot your Arch Linux installation entirely from RAM via GRUB.

## Architecture

```
GRUB
├── Arch Linux (disk)   ← normal use, updates, full 32GB RAM
└── Arch from RAM       ← maximum speed, writes go to tmpfs overlay
         ↑
    built from regular Arch → mksquashfs → .sfs file on disk
```

## How it works

1. `arch2ram-create` snapshots your running Arch system into a squashfs image
2. A custom initrd mounts the squashfs + overlayfs (writes to tmpfs) at boot
3. `switch_root` into the RAM copy — systemd starts normally
4. To update: boot into disk Arch, run `arch2ram-update`, reboot into RAM

## Files

```
initrd/          - custom initrd source (busybox + init script)
scripts/
  arch2ram-create   - build squashfs from running system
  arch2ram-update   - update existing squashfs
  arch2ram-install  - install GRUB entry + initrd to /boot
grub/            - GRUB menuentry template
docs/            - notes and decisions
```

## Requirements

- 32GB RAM (16GB allocated to RAM-OS, 16GB free for processes)
- Arch Linux on disk (the source system)
- GRUB bootloader
- `squashfs-tools` package

## Status

- [x] Proof of concept: custom initrd boots Alpine from RAM
- [ ] Build initrd with Arch busybox + losetup
- [ ] arch2ram-create script
- [ ] arch2ram-install script
- [ ] Full Arch boot from RAM
