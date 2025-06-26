---
title: PhotoRec
draft: false
tags: 
  - data
  - recovery
  - file carving
  - forensics
description: How to use PhotoRec for file recovery and carving from damaged drives and images.
date: 2025-06-25
aliases:
  - photorec
  - file recovery
  - file carving
---

# File Recovery with `photorec` on Ubuntu

This guide covers using `photorec` on Ubuntu and other Debian-based systems for recovering deleted files, photos, documents, and other data from damaged or corrupted storage media. PhotoRec specializes in file carving - extracting files directly from raw data without relying on filesystem metadata.

---

## 💡 Why Use `photorec`?

`photorec` is a powerful file carving tool that can recover files even when the filesystem is severely damaged or missing. It works by scanning for file signatures and reconstructing files from their raw data blocks.

### Best Use Cases
- **Recover deleted files** after accidental deletion or formatting
- **Extract files from corrupted filesystems** where the file allocation table is damaged
- **Carve files from disk images** created with tools like `ddrescue`
- **Recover data from memory cards, USB drives, and cameras**
- **Extract specific file types** (photos, documents, archives) from raw data

### When Not to Use
- When the filesystem is intact and files are just hidden (use file manager or `ls -la`)
- On physically failing drives without imaging first — use `ddrescue` to create an image
- When you need to preserve original filenames and directory structure (try `testdisk` first)

> [!tip]
> Always work on a disk image created with `ddrescue` rather than directly on a failing drive. This prevents further damage and allows multiple recovery attempts.

---

## 🧭 How It Works

`photorec` scans storage media byte-by-byte looking for known file signatures (magic numbers) that indicate the start of specific file types. It then attempts to reconstruct complete files by following the file's internal structure.

**Supported file types include:**
- **Images**: JPEG, PNG, GIF, TIFF, RAW formats (CR2, NEF, etc.)
- **Documents**: PDF, DOC/DOCX, XLS/XLSX, PPT/PPTX, ODT, RTF
- **Archives**: ZIP, RAR, 7Z, TAR, GZ
- **Media**: MP3, MP4, AVI, MOV, FLAC, OGG
- **And 400+ other formats**

---

## ⚙️ Installation

PhotoRec comes bundled with TestDisk:

```bash
sudo apt update && sudo apt install testdisk
```

---

## 🔍 Basic Usage

### 1. Create a Recovery Directory

Always recover files to a different drive to avoid overwriting data:

```bash
mkdir ~/recovered_files
```

### 2. Launch PhotoRec

```bash
sudo photorec
```

### 3. Navigate the Interface

1. **Select the disk/image** - Choose your source device or image file
2. **Select partition** - Choose the partition to scan (or `[Whole disk]` for complete scan)
3. **Choose filesystem type** - Usually `[Other]` for ext2/ext3/ext4, NTFS, FAT
4. **Select file types** - Choose specific formats or `[All]` for everything
5. **Choose destination** - Select your recovery directory
6. **Start recovery** - PhotoRec will begin carving files

---

## 📊 Working with Disk Images

### Creating an Image First (Recommended)

If your drive is showing signs of failure, create an image with `ddrescue`:

```bash
# Create a complete disk image
sudo ddrescue -f -n /dev/sdX backup.img backup.log

# For damaged drives, use more aggressive options
sudo ddrescue -d -A /dev/sdX backup.img backup.log
```

For more details on imaging, see our [[GNU ddrescue]] guide.

### Running PhotoRec on Images

```bash
# Launch PhotoRec and select your image file
sudo photorec backup.img
```

---

## 🎯 Command Line Usage

For automated or scripted recovery:

```bash
# Basic command line recovery
sudo photorec /log /d ~/recovered_files /dev/sdX1

# Recover specific file types only
sudo photorec /log /d ~/recovered_files /cmd /dev/sdX1 options,search,jpg+png+pdf

# Work with disk images
sudo photorec /log /d ~/recovered_files backup.img
```

### Command Line Options

- `/log` - Enable logging
- `/d destination` - Set recovery destination
- `/cmd device` - Specify source device/image
- `options,search,filetype` - Specify file types (jpg+png+pdf+doc, etc.)

---

## 🔧 Advanced Recovery Strategies

### Selective File Type Recovery

When you know what you're looking for:

```bash
# Photos only
sudo photorec /log /d ~/photos /cmd /dev/sdX1 options,search,jpg+png+gif+tiff+raw

# Documents only  
sudo photorec /log /d ~/documents /cmd /dev/sdX1 options,search,pdf+doc+docx+xls+xlsx+ppt+pptx

# Archives and compressed files
sudo photorec /log /d ~/archives /cmd /dev/sdX1 options,search,zip+rar+7z+tar+gz
```

### Large Drive Recovery

For very large drives, consider:

1. **Partition-by-partition recovery** instead of whole disk
2. **Specific file type targeting** to reduce processing time
3. **Using multiple recovery sessions** for different file types

---

## 📁 Post-Recovery File Organization

PhotoRec recovers files with generic names (f001.jpg, f002.pdf, etc.). Use these tools to organize:

```bash
# Sort by file type
mkdir ~/sorted_recovery
find ~/recovered_files -name "*.jpg" -exec cp {} ~/sorted_recovery/images/ \;
find ~/recovered_files -name "*.pdf" -exec cp {} ~/sorted_recovery/documents/ \;

# Use exiftool for photos (install with: sudo apt install exiftool)
exiftool -d "%Y/%m" "-directory<CreateDate" ~/sorted_recovery/images/*.jpg
```

---

## 🚨 Common Issues and Solutions

### "No file found, filesystem may be damaged"
- Try scanning the whole disk instead of individual partitions
- The filesystem metadata may be completely destroyed
- Consider using `testdisk` first to rebuild partition tables

### Recovery is very slow
- Working on a failing drive — create an image with `ddrescue` first
- Large drive with many small files — normal behavior
- Consider targeting specific file types only

### Files are corrupted after recovery
- Source drive may have physical damage
- Try different file type detection settings
- Some fragmentation is normal with severely damaged drives

### Running out of space during recovery
- Monitor disk space: `df -h`
- Recover to external storage with sufficient space
- Use selective file type recovery to prioritize important data

---

## 🔄 Integration with Other Tools

### Workflow with TestDisk

1. **First**: Try [[testdisk]] to repair partition tables and filesystem metadata
2. **Then**: Use PhotoRec for file carving if TestDisk cannot recover the structure
3. **Finally**: Manually sort and verify recovered files

### Workflow with ddrescue

1. **First**: Create a disk image with [[GNU ddrescue]] (especially for failing drives)
2. **Then**: Run PhotoRec on the image file
3. **Advantage**: Multiple recovery attempts without further damaging the original drive

---

## 🎯 Example Recovery Scenarios

### Scenario 1: Accidentally Formatted SD Card

```bash
# Quick recovery for photos
sudo photorec /log /d ~/camera_recovery /cmd /dev/sdX1 options,search,jpg+png+raw
```

### Scenario 2: Corrupted External Drive

```bash
# First, create an image
sudo ddrescue -f /dev/sdX external_backup.img external.log

# Then recover files from image
sudo photorec /log /d ~/external_recovery external_backup.img
```

### Scenario 3: Deleted Document Recovery

```bash
# Target office documents specifically
sudo photorec /log /d ~/document_recovery /cmd /dev/sdX1 options,search,pdf+doc+docx+xls+xlsx+ppt+pptx+odt
```

---

## ✅ Verification and Quality Check

After recovery:

1. **Check file counts and sizes**:
   ```bash
   find ~/recovered_files -type f | wc -l  # Count files
   du -sh ~/recovered_files                # Total size
   ```

2. **Test file integrity**:
   ```bash
   # Test image files
   identify ~/recovered_files/recup_dir.1/*.jpg
   
   # Test PDF files  
   pdfinfo ~/recovered_files/recup_dir.1/*.pdf
   ```

3. **Look for fragmented files** - Some files may be incomplete due to drive damage

---

## 📝 Best Practices

- **Always work on copies/images** when possible
- **Recover to a different drive** to avoid overwriting data
- **Start with specific file types** if you know what you need
- **Keep detailed logs** of your recovery attempts
- **Verify recovered files** before deleting originals
- **Consider professional data recovery** for critical data on severely damaged drives

---

**Related Guides:**
- [[testdisk]] - For partition and filesystem repair
- [[GNU ddrescue]] - For creating disk images before recovery
- [[dislocker]] - For accessing BitLocker-encrypted drives

---

*Need to recover from a BitLocker-encrypted drive? Utilize the Bitlocker Partition FUSE - [[dislocker]] to first unlock the volume.*