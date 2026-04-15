# Boot Flow

## Normal boot (disk)
```
GRUB → vmlinuz-linux-zen → mkinitcpio initrd → systemd → Arch
```

## RAM boot
```
GRUB → arch2ram-vmlinuz → arch2ram-initrd.gz
         ↓
       /init (ash script)
         ↓
       mount ext4 root (ro) → find arch-ram.sfs
         ↓
       mount squashfs (ro) at /squash_base
         ↓
       tmpfs overlay (rw writes go to RAM)
         ↓
       overlayfs at /new_root
         ↓
       switch_root /new_root /sbin/init
         ↓
       systemd → full Arch desktop
```

## Update workflow
```
1. Boot into disk Arch
2. sudo pacman -Syu       (update system)
3. sudo arch2ram-create   (rebuild squashfs, ~5 min)
4. reboot → RAM Arch      (fresh updated image)
```

## Why squashfs + overlayfs?
- squashfs: compressed (zstd), 100GB system → ~3-5GB image
- overlayfs: all writes go to tmpfs (RAM) — system feels like normal rw
- On reboot: writes are lost (RAM). /home stays on disk (not in image).

## RAM allocation (32GB system)
- squashfs image in RAM: ~4-6GB
- overlayfs tmpfs: 512MB
- process RAM: ~25GB free for applications
```
