# 🔍 SearchSploit Cheat Sheet

> Dành cho Pentesters & CTF Players — theo phong cách IppSec / HackTheBox / OSCP

---

## 📦 Cài đặt & Cập nhật

```bash
# Kali Linux (có sẵn trong exploitdb package)
sudo apt update && sudo apt install exploitdb

# Cập nhật database mới nhất (QUAN TRỌNG — làm trước khi dùng)
searchsploit -u

# Kiểm tra version
searchsploit --version
```

---

## 🔎 Tìm kiếm cơ bản

```bash
# Tìm theo tên phần mềm
searchsploit apache

# Tìm theo tên + version (cách phổ biến nhất của IppSec)
searchsploit apache 2.4

# Tìm nhiều từ khóa (AND logic)
searchsploit "windows" "smb" "remote"

# Tìm theo CVE
searchsploit CVE-2021-41773

# Tìm chính xác theo title (dùng dấu ngoặc kép)
searchsploit "Apache Struts 2"
```

---

## 🎯 Filter & Refine (hay dùng trong CTF)

```bash
# Chỉ hiện exploit (bỏ qua DoS, PoC kém)
searchsploit --type exploit apache

# Loại bỏ kết quả không liên quan (exclude)
searchsploit apache | grep -v "Ruby\|PHP"

# Tìm webapps riêng
searchsploit -t "webapps" wordpress 5.0

# Tìm local privilege escalation (rất dùng trong CTF post-exploit)
searchsploit linux kernel "local privilege escalation"
searchsploit "linux/local" sudo

# Tìm Windows PrivEsc
searchsploit windows "local" "privilege escalation" 2008
```

---

## 📋 Xem chi tiết & Copy exploit

```bash
# Xem nội dung exploit ngay trong terminal (không cần mở file)
searchsploit -x exploits/linux/remote/12345.py

# Copy exploit vào thư mục hiện tại (cách IppSec hay làm)
searchsploit -m exploits/linux/remote/12345.py

# Copy nhiều file cùng lúc
searchsploit -m exploits/linux/remote/12345.py exploits/windows/local/67890.c

# Tìm đường dẫn đầy đủ của file
searchsploit -p exploits/linux/remote/12345.py
```

---

## 🧰 Output Format (dùng trong script / pipeline)

```bash
# Output JSON (dùng để parse với jq hoặc Python)
searchsploit --json apache 2.4 | jq .

# Output JSON và lọc chỉ lấy tên + path
searchsploit --json "vsftpd 2.3.4" | jq '.RESULTS_EXPLOIT[] | {title: .Title, path: .Path}'

# Output không màu (dùng khi pipe sang file)
searchsploit --colour apache > results.txt

# Hiện full path (hữu ích khi cần biết vị trí file)
searchsploit -w apache 2.4
# -w hiện URL Exploit-DB thay vì local path
```

---

## 🔗 Kết hợp với Nmap (workflow chuẩn HackTheBox / OSCP)

```bash
# Scan với nmap lưu output XML
nmap -sV -sC -oX scan.xml 10.10.10.x

# Feed trực tiếp vào searchsploit (IppSec dùng thường xuyên)
searchsploit --nmap scan.xml

# Kết hợp với grep để lọc
searchsploit --nmap scan.xml | grep -i "remote\|rce\|exec"
```

---

## ⚡ Workflow thực chiến của IppSec / Top CTF Players

### Bước 1 — Enum xong, biết service + version

```bash
# Ví dụ: phát hiện vsftpd 2.3.4
searchsploit vsftpd 2.3.4
# → Thấy "Backdoor Command Execution" → copy về
searchsploit -m exploits/unix/remote/17491.rb
```

### Bước 2 — Không tìm thấy gì, mở rộng từ khóa

```bash
# Bỏ version, tìm tên chung
searchsploit vsftpd

# Thử tên khác nhau
searchsploit "ftp" "backdoor"
```

### Bước 3 — Đọc code trước khi chạy

```bash
# LUÔN đọc exploit trước khi chạy (tránh malicious PoC)
searchsploit -x exploits/unix/remote/17491.rb

# Hoặc mở bằng editor
searchsploit -p exploits/unix/remote/17491.rb
# → copy path → cat / vim path đó
```

### Bước 4 — Modify & Run

```bash
# Copy về, đổi LHOST/LPORT/RHOST
searchsploit -m exploits/unix/remote/17491.rb
vim 17491.rb
# Sửa RHOST, RPORT...
ruby 17491.rb
```

---

## 🔐 Tìm PrivEsc sau khi có shell (Post-Exploitation)

```bash
# Linux kernel local privesc (điền kernel version từ `uname -r`)
searchsploit linux kernel 4.4 "local privilege"
searchsploit linux "4.4.0" local

# Sudo privesc
searchsploit sudo 1.8

# SUID binary (ví dụ: tìm thấy /usr/bin/screen SUID)
searchsploit screen 4.5

# Windows PrivEsc (sau khi biết OS version từ systeminfo)
searchsploit "windows server 2008" "local"
searchsploit ms17-010
searchsploit ms14-058
```

---

## 🌐 Web Application Exploitation

```bash
# CMS phổ biến trong CTF
searchsploit wordpress 5.2
searchsploit drupal 7
searchsploit joomla 3.7

# Tìm SQLi / RCE cụ thể
searchsploit wordpress "sql injection"
searchsploit "remote code execution" php

# Tìm file upload bypass
searchsploit "file upload" php "remote"
```

---

## 🛠 Các flag quan trọng — Quick Reference

|Flag|Mô tả|Ví dụ|
|---|---|---|
|`-u`|Update database|`searchsploit -u`|
|`-m`|Mirror/copy exploit về CWD|`searchsploit -m path/to/exploit`|
|`-x`|Examine (xem nội dung)|`searchsploit -x path/to/exploit`|
|`-p`|Print full path|`searchsploit -p path/to/exploit`|
|`-w`|Hiện URL Exploit-DB|`searchsploit -w apache`|
|`--json`|Output JSON|`searchsploit --json vsftpd`|
|`--nmap`|Parse Nmap XML|`searchsploit --nmap scan.xml`|
|`--colour`|Disable màu (pipe-friendly)|`searchsploit --colour apache`|
|`-t`|Tìm theo title only|`searchsploit -t "Apache 2.4"`|
|`--type`|Filter theo loại exploit|`searchsploit --type webapps`|
|`-e`|Tìm exact string|`searchsploit -e "Apache 2.4.49"`|
|`--exclude`|Loại trừ từ khóa|`searchsploit apache --exclude="dos"`|

---

## 📁 Đường dẫn database mặc định

```bash
# Exploit-DB local path (Kali)
/usr/share/exploitdb/

# File exploits
/usr/share/exploitdb/exploits/

# File shellcodes
/usr/share/exploitdb/shellcodes/

# File CSV chứa metadata (để tự grep thủ công)
/usr/share/exploitdb/files_exploits.csv
/usr/share/exploitdb/files_shellcodes.csv

# Tự grep CSV khi searchsploit không đủ linh hoạt
grep -i "apache 2.4" /usr/share/exploitdb/files_exploits.csv
```

---

## 💡 Tips & Tricks từ CTF Community

```bash
# 1. Dùng grep sau searchsploit để lọc nhanh
searchsploit apache | grep -i "rce\|remote code\|exec"

# 2. Khi version không khớp, thử range
searchsploit "apache 2.4" | grep -E "2\.4\.(4[0-9]|5[0-9])"

# 3. Tìm shellcode cho kiến trúc cụ thể
searchsploit --shellcodes linux x86

# 4. Combine với MSF (Metasploit)
# Tìm trong searchsploit xong search lại trong msfconsole
searchsploit vsftpd 2.3.4
# → msfconsole → search vsftpd

# 5. So sánh với online Exploit-DB
# Thêm -w để lấy link, rồi mở browser xem PoC đầy đủ
searchsploit -w "vsftpd 2.3.4"

# 6. Tìm theo platform
searchsploit -t "windows/remote" iis
searchsploit -t "linux/local" kernel

# 7. Khi làm OSCP/HTB — luôn check cả shellcodes
searchsploit --shellcodes windows x86_64
```

---

## ⚠️ Lưu ý quan trọng

- **Luôn `searchsploit -u` trước khi làm việc** — database thay đổi liên tục
- **Đọc code exploit (`-x`) trước khi chạy** — tránh malicious PoC
- **Kiểm tra chéo với Exploit-DB online** — local DB đôi khi lag sau online vài ngày
- **Một số exploit cần modify** — đổi LHOST, RHOST, port trước khi chạy
- **PoC != Working Exploit** — nhiều entry chỉ là proof-of-concept, cần adapt thêm

---

