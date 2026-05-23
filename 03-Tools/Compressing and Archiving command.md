# 📦 Linux Compressing & Archiving Cheatsheet

---

## tar — Tape Archive

### Create Archives

|Command|Description|
|---|---|
|`tar -cvf archive.tar file/`|Create `.tar` (no compression)|
|`tar -czvf archive.tar.gz file/`|Create `.tar.gz` (gzip)|
|`tar -cjvf archive.tar.bz2 file/`|Create `.tar.bz2` (bzip2)|
|`tar -cJvf archive.tar.xz file/`|Create `.tar.xz` (xz)|
|`tar -czvf archive.tar.gz -C /path/ .`|Archive from specific directory|

### Extract Archives

|Command|Description|
|---|---|
|`tar -xvf archive.tar`|Extract `.tar`|
|`tar -xzvf archive.tar.gz`|Extract `.tar.gz`|
|`tar -xjvf archive.tar.bz2`|Extract `.tar.bz2`|
|`tar -xJvf archive.tar.xz`|Extract `.tar.xz`|
|`tar -xzvf archive.tar.gz -C /output/`|Extract to specific directory|

### Inspect Archives

|Command|Description|
|---|---|
|`tar -tvf archive.tar`|List contents without extracting|
|`tar -tzvf archive.tar.gz`|List contents of `.tar.gz`|
|`tar -xzvf archive.tar.gz file.txt`|Extract a single file|

### tar Flags Reference

|Flag|Meaning|
|---|---|
|`-c`|Create archive|
|`-x`|Extract archive|
|`-t`|List contents|
|`-v`|Verbose output|
|`-f`|Specify filename|
|`-z`|Use gzip compression|
|`-j`|Use bzip2 compression|
|`-J`|Use xz compression|
|`-C`|Change to directory|
|`--exclude=`|Exclude files/dirs|

---

## gzip / gunzip

|Command|Description|
|---|---|
|`gzip file.txt`|Compress → `file.txt.gz` (removes original)|
|`gzip -k file.txt`|Compress and **keep** original|
|`gzip -d file.txt.gz`|Decompress|
|`gunzip file.txt.gz`|Decompress (same as above)|
|`gzip -r folder/`|Recursively compress all files in folder|
|`gzip -l file.gz`|Show compression info|
|`gzip -9 file.txt`|Max compression level (1=fast, 9=best)|
|`zcat file.gz`|View content without decompressing|

---

## bzip2 / bunzip2

|Command|Description|
|---|---|
|`bzip2 file.txt`|Compress → `file.txt.bz2`|
|`bzip2 -k file.txt`|Compress and keep original|
|`bzip2 -d file.txt.bz2`|Decompress|
|`bunzip2 file.txt.bz2`|Decompress (same as above)|
|`bzcat file.bz2`|View content without decompressing|
|`bzip2 -9 file.txt`|Max compression level|

---

## xz

|Command|Description|
|---|---|
|`xz file.txt`|Compress → `file.txt.xz`|
|`xz -k file.txt`|Compress and keep original|
|`xz -d file.txt.xz`|Decompress|
|`unxz file.txt.xz`|Decompress (same as above)|
|`xzcat file.xz`|View content without decompressing|
|`xz -9 file.txt`|Max compression level|
|`xz -T 4 file.txt`|Use 4 threads for compression|

---

## zip / unzip

|Command|Description|
|---|---|
|`zip archive.zip file1 file2`|Zip specific files|
|`zip -r archive.zip folder/`|Zip entire folder recursively|
|`zip -e archive.zip file.txt`|Zip with password encryption|
|`zip -9 archive.zip file.txt`|Max compression level|
|`unzip archive.zip`|Extract zip|
|`unzip archive.zip -d /output/`|Extract to specific directory|
|`unzip -l archive.zip`|List contents without extracting|
|`unzip -p archive.zip file.txt`|Print file to stdout|

---

## 7zip (7z)

|Command|Description|
|---|---|
|`7z a archive.7z file/`|Create `.7z` archive|
|`7z x archive.7z`|Extract with full paths|
|`7z e archive.7z`|Extract all to current dir|
|`7z l archive.7z`|List contents|
|`7z t archive.7z`|Test archive integrity|
|`7z a -p archive.7z file/`|Create with password|
|`7z a -mx=9 archive.7z file/`|Max compression level|
|`7z a archive.zip file/`|Create zip using 7z|

---

## Compression Format Comparison

|Format|Extension|Speed|Ratio|Use Case|
|---|---|---|---|---|
|gzip|`.gz`|⚡⚡⚡ Fast|Medium|General use, logs|
|bzip2|`.bz2`|⚡⚡ Medium|Good|Source code|
|xz|`.xz`|⚡ Slow|**Best**|Distributions, backups|
|zip|`.zip`|⚡⚡⚡ Fast|Medium|Cross-platform sharing|
|7z|`.7z`|⚡ Slow|**Best**|Maximum compression|
|lz4|`.lz4`|⚡⚡⚡⚡|Low|Real-time, databases|

---

## Penetration Testing Use Cases

```bash
# Exfiltrate data — compress before transfer
tar -czvf loot.tar.gz /home/user/documents/
split -b 5M loot.tar.gz loot_part_   # split into 5MB chunks

# Quickly archive entire web directory
tar -czvf backup.tar.gz /var/www/html/

# Exclude sensitive dirs when archiving
tar -czvf archive.tar.gz /etc/ --exclude=/etc/shadow --exclude=/etc/ssl

# Fast compress large log files
gzip -9 -k /var/log/auth.log

# Password-protected zip for findings report
zip -e -r pentest_report.zip ./findings/

# Check archive integrity
tar -tzvf archive.tar.gz > /dev/null && echo "OK" || echo "CORRUPTED"

# Extract only specific file type from archive
tar -xzvf archive.tar.gz --wildcards '*.txt'

# Create archive and pipe over SSH (no temp file)
tar -czvf - /data/ | ssh user@remote "cat > backup.tar.gz"

# Decompress unknown archive type
file archive.*       # identify type first
```

---

## Identify Compressed File Type

```bash
file archive.bin        # detect actual file type
xxd archive.bin | head  # check magic bytes

# Common magic bytes:
# 1f 8b       → gzip
# 42 5a 68    → bzip2 (BZh)
# fd 37 7a    → xz
# 50 4b 03 04 → zip (PK)
# 37 7a bc af → 7z
```

---

## Multi-Part Archives (Split & Combine)

### Split a file into parts

|Command|Description|
|---|---|
|`zip -s 5m archive.zip folder/`|Create split zip (5MB per part) → `.z01, .z02, .zip`|
|`split -b 5M file.zip part_`|Split any file into 5MB chunks → `part_aa, part_ab, ...`|
|`split -n 3 file.zip part_`|Split into exactly 3 parts|

### Combine parts back into one file

|Command|Description|
|---|---|
|`cat file.z01 file.z02 file.zip > combined.zip`|Combine `.z01/.z02/.zip` parts (order matters!)|
|`cat part_* > combined.zip`|Combine `part_aa, part_ab, ...` parts|
|`cat part1 part2 part3 > combined.zip`|Combine explicitly named parts|
|`zip -FF file.zip --out combined.zip`|Repair & merge split zip|
|`7z x file.z01`|Auto-detect and extract all parts|

> ⚠️ **Order matters:** The last part (`.zip`) must always come last — it contains the central directory.

### Verify after combining

```bash
file combined.zip                  # confirm it's a valid zip
xxd combined.zip | head -2         # magic bytes must start with PK (50 4b)
unzip -t combined.zip              # test integrity
unzip -l combined.zip              # list contents
```

### Extract combined zip

```bash
unzip combined.zip                 # normal extract
unzip -P supersecret combined.zip  # extract with password
7z x combined.zip                  # extract with 7z
```

---

## Quick Reference

```bash
# Create
tar -czvf out.tar.gz folder/    # tar + gzip
zip -r out.zip folder/          # zip

# Extract
tar -xzvf file.tar.gz           # tar.gz
tar -xjvf file.tar.bz2          # tar.bz2
tar -xJvf file.tar.xz           # tar.xz
unzip file.zip                  # zip
7z x file.7z                    # 7z

# Inspect
tar -tzvf file.tar.gz           # list tar.gz
unzip -l file.zip               # list zip
7z l file.7z                    # list 7z
```

---
