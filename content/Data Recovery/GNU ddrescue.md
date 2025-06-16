---
title: ddrescue (GNU)
draft: false
tags:
  - data
  - recovery
  - imaging
description: How to use / perform data recovery with ddrescue
date: 2025-06-14
aliases:
  - ddrescue
  - drive imaging
---
# Data Recovery with `ddrescue` on Ubuntu

This guide explains how and why to use `ddrescue` on Ubuntu or other Debian-based systems to recover data from problematic storage media. It focuses on real-world usage, signs of failure, proper command usage depending on the situation, and practical recovery strategies.

---

## 💡 Why Use `ddrescue`?

`ddrescue` excels in scenarios where other tools would fail or worsen the situation. It is designed for *data preservation*, especially in the face of hardware degradation or logical corruption.

### Best Use Cases
- **Failing hard drives or SSDs** (with bad sectors or dying controllers)
- **Drives with corrupted filesystems** (won't mount, read errors, etc.)
- **Clone-before-fix workflow**: Always clone a drive before attempting repairs (fsck, testdisk, etc.)
- **Maximizing data salvage**: You can resume recovery later, thanks to logging.

### When Not to Use
- On fully working drives (use `dd` instead for speed and simplicity)
- Physically damaged drives (e.g. burning smell, loud clicking) — stop immediately and consider a data recovery lab

---

## 🧭 How It Works

`ddrescue` copies data from a source to a target, intelligently handling read errors and retrying failed regions in multiple passes. It maintains a map of what data was read successfully and what wasn’t, via a log file.

- **First Pass**: Fast read, skips over unreadable areas
- **Second Pass**: Tries to recover skipped areas with more effort
- **Final Pass**: Aggressively retries the most damaged sectors (optional)

---

## 🔍 Finding the Right Device with `lsblk`

Always verify the correct source and destination before running anything. Use:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,MODEL,SERIAL
```

Check that your source (e.g. `/dev/sdb`) is **unmounted** and **matches the drive you're recovering**. Compare model/serial numbers to avoid mistakes.

> [!warning] Never operate on a mounted or incorrect device. You risk destroying your data.

---

## ⚙️ Installation

Install via APT:

```bash
sudo apt update && sudo apt install gddrescue
```

> [!note] The package is named `gddrescue` but the command is `ddrescue`.

---

## 🧪 Flag Overview and Recovery Strategies

### Most Common Flags

| Flag        | Description                                                              |
| ----------- | ------------------------------------------------------------------------ |
| `-f`        | Forces writing to output file (required if it exists or is a raw device) |
| `-n`        | Skip error retries — good for first pass (fast)                          |
| `-r N`      | Retry bad blocks N times (e.g. `-r3`)                                    |
| `-d`        | Use direct disk access — bypasses OS cache (may help on failing drives)  |
| `--ask`     | Prompts before overwriting output (useful for safety)                    |
| `--idirect` | Use direct I/O on input only                                             |

---

## 🔁 Common Command Sequences

### 1. First Pass (non-invasive, fast)
```bash
sudo ddrescue -f -n /dev/sdX ./drive_backup.img ./drive_backup.log
```
- Skips errors
- Avoids hammering weak areas

### 2. Retry Failed Areas
```bash
sudo ddrescue -d -r3 /dev/sdX ./drive_backup.img ./drive_backup.log
```
- Try recovering bad sectors
- May freeze temporarily on unreadable sectors — **watch `dmesg`**

> [!note]
> This is the most common command you can utilize for generic drive failure.

### 3. Final Attempt (max recovery) - CAREFUL WITH THIS ONE PLEASE
```bash
sudo ddrescue -d -r-1 /dev/sdX ./drive_backup.img ./drive_backup.log
```
- Infinite retries on bad blocks (use with care)
- Watch drive temperature and noise
> [!danger] Please refrain from using this command unless you are confident on usage and physical condition of device.

---

## 🛑 When to Stop Recovery Early

Signs that you should stop the recovery and assess:

- **dmesg shows I/O errors, kernel resets, or controller hangs**
- **Drive starts disconnecting or vanishing from `lsblk`**
- **Audible clicks** increase or drive temperature spikes (>50°C)
- **The recovery stalls at 0 B/s for a long period**

Stop and reassess — further attempts may worsen the drive.

---

## 📦 Image Handling

You can inspect or recover data from the image file after successful extraction:

### Mount the image read-only
```bash
sudo mount -o loop,ro ./drive_backup.img /mnt/recovery
```

### Use file recovery tools
- `photorec` — file carving
- [[testdisk]] — partition and boot sector recovery
- `autopsy` — forensic GUI

---

## 🔐 Important Notes

- Always use a **log file** to resume or refine the recovery.
- Only write the image back to disk if you're **sure it's safe**.
- Drives with rising **SMART Reallocated Sectors** may be degrading fast — act quickly.

---

## 🔗 Related

- [[Filesystem Repair with fsck]]
- [[Mounting Disk Images with loopback]]
- [[SMART Disk Health Monitoring]]
