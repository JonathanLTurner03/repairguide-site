---
title: TestDisk
draft: false
tags: 
description: How to use / perform data recovery with testdisk.
date: 2025-06-14
aliases:
  - photorec
---
# Data Recovery with `testdisk` on Ubuntu

This guide provides a focused overview of using `testdisk` on Ubuntu or other Debian-based systems for partition and boot sector recovery. It covers how it works, when to use it, recommended workflows, how to analyze filesystems, how to assign partition types, and how to recognize signs of deeper disk failure.

---

## 💡 Why Use `testdisk`?

`testdisk` is a forensic recovery tool built for fixing partition tables and recovering lost or deleted partitions. Unlike `ddrescue`, it does not clone the drive but works directly with the metadata and filesystem structure.

### Best Use Cases
- **Recover deleted partitions** (especially on MBR, GPT, or Apple partitions)
- **Fix corrupted partition tables** that prevent booting or mounting
- **Repair FAT/NTFS/EXT boot sectors** and backup sector mismatches
- **Find lost partitions on cloned `.img` files**

### When Not to Use
- When the drive is physically failing — use `ddrescue` to image it first
- If you’ve already written new partitions or formatted the drive (may overwrite recoverable data)

> [!tip]
> Always make a clone image with `ddrescue` before attempting fixes with `testdisk`.

---

## 🧭 How It Works

`testdisk` scans the disk’s low-level structure for signs of valid partitions and filesystem headers. It compares against the current partition table and allows you to restore missing entries or repair inconsistencies.

It supports:
- MBR, GPT, APM, BSD, SUN, and more
- FAT12/FAT16/FAT32, NTFS, EXT2/3/4, HFS, exFAT

---

## ⚙️ Installation

Install via APT:

```bash
sudo apt update && sudo apt install testdisk
```

Run with:
```bash
sudo testdisk
```

> [!info]
> `testdisk` runs in an ncurses (text) interface. Navigation is keyboard-based.

---

## 🔁 Recommended Recovery Workflow

1. **Select Disk** (e.g., `/dev/sdb` or image file)
2. **Select Partition Table Type** (usually auto-detected)
3. Choose **[Analyse]** to scan for partitions
4. Use **[Quick Search]** for a fast scan
5. If missing partitions aren't shown, try **[Deeper Search]**
6. Review found partitions — press `P` to list files and verify contents
7. **Assign Partition Types**:
   - Use the **arrow keys** to highlight a partition
   - Use the following keys to change type:
     - `P` – Primary
     - `L` – Logical
     - `D` – Deleted (ignored on write)
     - `*` – Toggle bootable flag
   - Changes appear on the left (e.g. `P` for Primary)
   - You can inspect files in a partition by highlighting it and pressing `P`
8. When ready, select **[Write]** to save the new partition table

> [!warning]
> Writing partition changes is destructive — confirm everything is correct first. Use the file listing tool (`P`) to preview contents before restoring.

---

## 🔎 Analyzing Filesystems and Listing Files

Once a partition is found and highlighted, press `P` to list its contents.

You can:
- Use arrow keys to browse
- Use `:` (colon) to **select multiple files or directories**
- Press `C` to **copy** selected items to a chosen destination
- Use `q` to go back or exit

> [!tip]
> When prompted for a destination path, use `..` to move up a directory, and Enter to select a folder.

> [!example]
> To recover a few specific folders, select them with `:` and press `C` to copy to `/mnt/recovery`.

### When to Use Deeper Search
- **Quick Search fails to find expected partitions**
- **Multiple OS installations or dual boot setups existed**
- **Overwritten bootloaders or corrupted partition tables**
- **Recent repartitioning or OS install wiped previous layouts**

> [!info]
> Deeper searches take time but can uncover partitions missed by Quick Search.

---

## 🧪 Example Recovery Workflow (ddrescue → testdisk)

Here’s a real-world example of recovering data from a failing Windows drive:

### 1. Clone the Failing Drive with `ddrescue`
```bash
sudo ddrescue -f -n /dev/sdX ./windows_backup.img ./windows_backup.log 
# or
sudo ddrescue -d -r3 /dev/sdX ./windows_backup.img ./windows_backup.log
```

### 2. Run `testdisk` on the Image File
```bash
sudo testdisk ./windows_backup.img
```

- Select the appropriate partition type (usually Intel or EFI/GPT)
- Use [Analyse] → [Quick Search] → [Deeper Search] if needed

### 3. Identify and List Partitions
- Use arrow keys to review each partition
- Press `P` on a likely NTFS partition to view files

### 4. Copy Files from NTFS
- Use `:` to select user folders like `Users/John/Documents`
- Press `C` and choose `/mnt/recovery` as the destination
- Files will be copied with original structure

> [!tip]
> If working from a live USB, mount a large external drive at `/mnt/recovery` first.

### 5. Post-Recovery
- Optionally mount the image: `sudo mount -o loop,ro ./windows_backup.img /mnt/tmp`
- Backup recovered data elsewhere
- Run `fsck` or `chkdsk /f` on the image only if planning to reuse

---

## 🛑 Signs You Should Stop or Re-Evaluate

> [!danger]
> Stop if you see these:
>
> - Partition tables keep resetting after reboot
> - SMART errors appear (`dmesg`, `smartctl`)
> - Drive is disconnecting, hanging, or overheating

In these cases, return to `ddrescue` and clone the drive before continuing.

---

## 🧼 Cleanups and Post-Recovery

Once the partition is restored:
- Mount it normally: `sudo mount /dev/sdX1 /mnt`
- Check filesystem health: `sudo fsck /dev/sdX1`
- Copy essential data immediately — the disk may still be unreliable

---

## 🔗 Related

- [[Data Recovery with ddrescue on Ubuntu/Debian]]
- [[SMART Disk Health Monitoring]]
- [[Mounting Disk Images with loopback]]
