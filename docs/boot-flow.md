# Boot Flow

## Normal boot (disk)
```
GRUB → /boot/vmlinuz-linux → /boot/initramfs-linux.img → systemd → Arch
```

## RAM boot — what GRUB hands the kernel
```
GRUB selects "Arch Linux from RAM":
    linux  /vmlinuz-linux archisobasedir=var/lib/arch2ram archisosearchuuid=<ROOT_UUID> copytoram=y cow_spacesize=2G
    initrd /intel-ucode.img /initramfs-arch2ram.img
```

`initramfs-arch2ram.img` is built by mkinitcpio with the archiso hook set —
the same hooks the official Arch install ISO uses.

## What the initramfs does (mkinitcpio-archiso hook)

```
[ udev brings up devices ]
        ↓
archiso_mount_handler reads boot params:
    archisobasedir=var/lib/arch2ram
    archisosearchuuid=<ROOT_UUID>
    copytoram=y
    cow_spacesize=2G
        ↓
_search_for_archisodevice
    resolves UUID=<ROOT_UUID> via blkid → finds /dev/sdXN
        ↓
_mnt_dev — mounts the partition read-only at /run/archiso/bootmnt
        ↓
_mnt_fs — picks /run/archiso/bootmnt/var/lib/arch2ram/x86_64/airootfs.sfs
    copytoram=y → cp the .sfs into a tmpfs at /run/archiso/copytoram
    (after this point, the partition can be unmounted — boot has no disk dep)
    losetup the .sfs and mount it at /run/archiso/airootfs (read-only squashfs)
        ↓
cowspace tmpfs (2 GiB) mounted at /run/archiso/cowspace
        ↓
_mnt_overlayfs
    lowerdir = /run/archiso/airootfs        (read-only squashfs)
    upperdir = /run/archiso/cowspace/upperdir
    workdir  = /run/archiso/cowspace/workdir
    → overlay mounted at /new_root
        ↓
init copies /run, releases the bootmnt mount, then:
    switch_root /new_root /sbin/init
        ↓
systemd boots normally on top of the overlay — your full Arch desktop, in RAM.
```

## Update workflow
```
1. Boot into disk Arch (normal entry).
2. sudo pacman -Syu                  (update normally)
3. sudo arch2ram-create              (rebuild the squashfs, ~5 min)
4. reboot → "Arch Linux from RAM"    (fresh updated image)
```

The initramfs (`initramfs-arch2ram.img`) is rebuilt automatically by the
mkinitcpio pacman hook whenever the kernel updates — you don't need to
re-run `arch2ram-install` for that.

## Why squashfs + overlayfs (and copytoram)?

- **squashfs** — compressed read-only filesystem. A 30 GB system compresses
  to ~5 GB with zstd. The kernel reads it on demand; with copytoram the
  source partition is touched once and never again.
- **overlayfs** — stacks a writable tmpfs over the read-only squashfs.
  Every write goes to RAM. The running system sees a normal `/`.
- **copytoram** — after copying the .sfs into tmpfs, the original
  partition can sit untouched. Read latency drops from NVMe (~3500 MB/s)
  to RAM (~40000 MB/s). On reboot, the tmpfs evaporates.

## RAM allocation (32 GB system, ~5 GB image)

| chunk                          | size      |
|--------------------------------|-----------|
| squashfs image in copytoram    | ~5 GB     |
| overlay cowspace (writes)      | 2 GB cap  |
| systemd + apps                 | ~5–8 GB   |
| **free for your processes**    | **~17 GB**|
