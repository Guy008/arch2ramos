# arch2ramos

> **Run your OS entirely from RAM — boot in seconds, zero disk I/O, maximum speed.**

This project was a years-long dream. This is the complete guide to making it real.

---

## Table of Contents

1. [The Idea](#the-idea)
2. [How It Works](#how-it-works)
3. [What We Proved](#what-we-proved)
4. [Requirements](#requirements)
5. [Installation Guide — Arch Linux from RAM](#installation-guide--arch-linux-from-ram)
6. [Installation Guide — Batocera from RAM](#installation-guide--batocera-from-ram)
7. [The Update Workflow](#the-update-workflow)
8. [GRUB Menu — Switching Between Systems](#grub-menu--switching-between-systems)
9. [Why This Works](#why-this-works)

---

## The Idea

Modern systems have 32GB of RAM. A full Linux desktop fits in 5-8GB.
**That leaves 24GB of RAM completely idle.**

What if you ran the OS itself from RAM?

- Boot in seconds (squashfs loads to RAM once)
- Zero disk I/O during runtime (everything in RAM)
- All writes go to a RAM overlay (tmpfs) — disk stays untouched
- Switch between disk-Arch and RAM-Arch from GRUB
- Update the RAM image from the disk system

This is not a VM. This is not a container.
The OS runs natively — kernel, systemd, everything — **from RAM**.

---

## How It Works

### The Stack

```
GRUB
├── Arch Linux (disk)      ← normal use, updates, all 32GB RAM free
└── Arch Linux (RAM)       ← maximum speed, writes to tmpfs
```

### The Boot Flow (RAM mode)

```
GRUB loads:
  arch2ram-vmlinuz    (the kernel)
  arch2ram-initrd.gz  (tiny initrd with ash + losetup)
         ↓
  initrd mounts root partition (read-only)
         ↓
  finds arch-ram.sfs (squashfs of your entire system)
         ↓
  mounts squashfs as read-only base layer
         ↓
  creates tmpfs overlay (writes go to RAM)
         ↓
  overlayfs = squashfs base + tmpfs writes = full r/w system
         ↓
  switch_root → /sbin/init → systemd → your desktop
```

### The Initrd

The initrd is built from **Alpine Linux minirootfs** — because Alpine uses
statically-linked musl busybox. This means it works reliably without depending
on any host libraries. It contains:

- `ash` shell
- `mount` with squashfs, overlay, loop support
- `losetup` for loop devices
- `switch_root` to hand off to the real OS
- `libc.musl-x86_64.so.1` + `ld-musl-x86_64.so.1`

---

## What We Proved

**April 2026 — Proof of concept successful.**

Custom initrd booted from GRUB, using Arch zen kernel + Alpine musl busybox:

```
=== Alpine RAM initrd - WORKING ===
Kernel: 6.19.11-zen1-1-zen
Mem:  31.1G total | 104.8M used | 30.9G free
~ #
```

104MB used. 30.9GB free. Shell running from RAM.
The foundation works. Now we build on it.

---

## Requirements

- **RAM:** 32GB (16GB for OS image, ~16GB free for your processes)
- **Disk:** Arch Linux installation (source system)
- **Bootloader:** GRUB
- **Package:** `squashfs-tools` (`sudo pacman -S squashfs-tools`)

---

## Installation Guide — Arch Linux from RAM

### Step 0: Fresh Arch install

Install Arch Linux normally. Set up everything you want in your RAM system:
desktop environment, packages, configs. This becomes the RAM image.

Recommended partition layout:
```
sda1  FAT32   512MB    /boot         (EFI + GRUB)
sda2  ext4    40GB+    /             (system + squashfs file lives here)
sda3  btrfs   rest     /home         (personal files — NOT in RAM image)
```

### Step 1: Clone this repo

```bash
git clone https://github.com/Guy008/arch2ramos.git
cd arch2ramos
```

### Step 2: Install (sets up initrd + GRUB entry)

```bash
sudo scripts/arch2ram-install
```

This will:
- Download Alpine minirootfs (3.4MB) as the initrd base
- Build `arch2ram-initrd.gz` and copy to `/boot`
- Copy kernel to `/boot/arch2ram-vmlinuz`
- Add "Arch Linux from RAM" entry to GRUB

### Step 3: Create the squashfs image

```bash
sudo scripts/arch2ram-create
```

This runs `mksquashfs /` and creates `/arch-ram.sfs` on your root partition.
Takes 3-8 minutes depending on system size. Typical output: 4-7GB (zstd compressed).

**What's excluded from the image:**
- `/home` — stays on disk, mounted normally
- `/proc`, `/sys`, `/dev`, `/run`, `/tmp`
- `/var/cache/pacman/pkg` — no need to RAM-boot the package cache
- `/var/log` — logs go to tmpfs in RAM mode

### Step 4: Reboot

```bash
reboot
```

In GRUB, select **"Arch Linux from RAM"**.

You'll see:
```
=== arch2ramos: booting Arch from RAM ===
[1/5] Mounting source partition...
[2/5] Mounting squashfs from RAM image...
[3/5] Setting up overlayfs (writes go to RAM)...
[4/5] Moving mounts into new root...
[5/5] Switching to Arch root...
```

Then systemd takes over and boots normally — but everything is in RAM.

---

## Installation Guide — Batocera from RAM

Batocera packages its OS as a squashfs inside a FAT32 partition inside a `.img.gz` file.
We exploit this directly.

### Step 1: Download Batocera

```bash
mkdir -p ~/batocera-ram
cd ~/batocera-ram
curl -L -o batocera.img.gz \
  "https://mirrors.o2switch.fr/batocera/x86_64/stable/last/batocera-x86_64-42-20251006.img.gz"
```

### Step 2: Decompress

```bash
zcat batocera.img.gz > batocera.img
# Result: ~11GB raw disk image
```

### Step 3: Install Batocera GRUB entry

```bash
sudo scripts/arch2ram-install --batocera ~/batocera-ram/batocera.img
```

This sets up a GRUB entry that:
1. Mounts your home partition (btrfs)
2. Sets up a loop device on `batocera.img`
3. Mounts the FAT32 BATOCERA partition from within the image
4. Mounts the squashfs + overlayfs
5. `switch_root` into Batocera

### Step 4: Reboot → "Batocera from RAM"

Batocera boots entirely from RAM. EmulationStation starts.
ROMs stay on disk/USB — only the OS is in RAM.

---

## The Update Workflow

### Update Arch RAM image

```bash
# 1. Boot into disk Arch
# 2. Do your updates
sudo pacman -Syu
# install new packages, change configs, etc.

# 3. Rebuild the RAM image
sudo arch2ramos/scripts/arch2ram-create

# 4. Reboot into RAM
reboot
```

### Update Batocera

```bash
cd ~/batocera-ram
# Download new version
curl -L -o batocera-new.img.gz "https://..."
zcat batocera-new.img.gz > batocera.img
# Next RAM boot uses the new image automatically
```

---

## GRUB Menu — Switching Between Systems

After installation, your GRUB will look like:

```
┌─────────────────────────────────────┐
│  Arch Linux                         │  ← disk, normal boot
│  Arch Linux (advanced options)      │
│  ─────────────────────────────────  │
│  Arch Linux from RAM            ←   │  ← RAM boot, maximum speed
│  Batocera from RAM              ←   │  ← gaming OS from RAM
│  ─────────────────────────────────  │
│  Windows Boot Manager               │
└─────────────────────────────────────┘
```

Switch anytime. Each system is fully independent.

**The bonus:** While running disk-Arch, you have all 32GB RAM for your processes.
While running RAM-Arch, the OS uses ~5-8GB, leaving ~24GB free.

---

## Why This Works

### RAM speed vs disk speed

| Storage | Read speed |
|---------|-----------|
| NVMe SSD | ~3,500 MB/s |
| RAM (DDR4) | ~40,000 MB/s |

RAM is **10x faster** than the fastest SSD for random reads.
Every file open, every library load, every config read — instant.

### squashfs + overlayfs

- **squashfs**: read-only compressed filesystem. Your 15GB system compresses to ~5GB.
  Mounted directly — no extraction needed.
- **overlayfs**: stacks a read-write tmpfs on top of the read-only squashfs.
  Writes go to RAM. Reads come from the squashfs base.
  The running system sees a normal read-write filesystem.

### Why not just tmpfs?

Extracting the entire system to tmpfs would take minutes and use more RAM.
squashfs stays compressed in RAM — the kernel decompresses pages on demand.
With zstd compression, decompression is near-instant.

### The initrd trick

Standard initrds extract themselves and hand off to the OS.
Ours does something different:
1. Mounts the disk partition (read-only)
2. Mounts the squashfs from a file on that partition
3. Creates an overlayfs
4. `switch_root` — the kernel switches to the new root and discards the initrd

The entire setup happens before systemd even starts.

---

## Project Structure

```
arch2ramos/
├── README.md                  ← you are here
├── initrd/
│   ├── init                   ← Arch boot script
│   └── init-batocera          ← Batocera boot script
├── scripts/
│   ├── arch2ram-create        ← build squashfs from running system
│   ├── arch2ram-install       ← install initrd + GRUB entry
│   └── arch2ram-update        ← update existing squashfs
├── grub/
│   └── menuentry.example      ← GRUB entry template
└── docs/
    └── boot-flow.md           ← detailed architecture notes
```

---

*Built from scratch, one layer at a time. From dream to working shell prompt.*
