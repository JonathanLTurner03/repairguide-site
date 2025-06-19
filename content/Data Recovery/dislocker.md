---
title: Dislocker - BitLocker FUSE
draft: false
tags: 
  - bitlocker
  - windows
  - encryption
  - fuse
description: How to access BitLocker-encrypted drives on Linux using dislocker and FUSE.
date: 2025-06-16
aliases:
  - bitlocker
  - dislocker
---

# Accessing BitLocker Drives with `dislocker` on Linux

`dislocker` allows you to access BitLocker-encrypted (Windows) volumes on Linux by mounting them via FUSE. This is essential for data recovery from drives protected by BitLocker.

---

## 💡 Why Use `dislocker`?

- **Read and write access** to BitLocker volumes from Linux
- **Essential for data recovery** when Windows is unavailable or unbootable
- Works with both **physical drives** and **disk images** (e.g., from `ddrescue`)

---

## ⚙️ Installation

Install via APT:

```bash
sudo apt update && sudo apt install dislocker
```

---

## 🔑 Usage Example

### 1. Identify the BitLocker Partition

Use `lsblk` or `fdisk -l` to find the encrypted NTFS partition (often labeled "Microsoft basic data").

### 2. Unlock the Partition

```bash
sudo mkdir /mnt/bitlocker /mnt/bitlocker_unlocked
sudo dislocker -V /dev/sdXn -uYourPassword -- /mnt/bitlocker
```
- Replace `/dev/sdXn` with your partition (e.g., `/dev/sdb2`)
- Use `-uYourPassword` for password, or `-k` for recovery key file

### 3. Mount the Unlocked Volume

```bash
sudo mount -o loop,ro /mnt/bitlocker/dislocker-file /mnt/bitlocker_unlocked
```
- Now `/mnt/bitlocker_unlocked` contains your files

---

## 🧪 Recovery Workflow (with ddrescue)

1. **Clone the drive** with [[ddrescue (GNU)]]
2. **Unlock the image** with `dislocker`:
   ```bash
   sudo dislocker -V ./windows_backup.img -uYourPassword -- /mnt/bitlocker
   ```
3. **Mount and recover files** as above

---

## 🔗 Related

- [[Data Recovery with ddrescue on Ubuntu]]
- [[TestDisk]]
- [Dislocker GitHub](https://github.com/Aorimn/dislocker)