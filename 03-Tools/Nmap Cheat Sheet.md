# 🗺️ Nmap Cheatsheet — Pentest Edition (IppSec Style)

> Tổng hợp các kỹ thuật Nmap thực chiến, theo workflow của IppSec và các pentester hàng đầu (HTB, OSCP, real-world engagements).

---

## ⚡ Quick Recon (IppSec Workflow)

```bash
# Bước 1 — Full TCP port scan nhanh (IppSec luôn bắt đầu bằng này)
nmap -p- --min-rate 10000 -oA scans/all-ports <IP>

# Bước 2 — Service/version scan trên các port tìm được
nmap -p 22,80,443 -sCV -oA scans/targeted <IP>

# Bước 3 — UDP scan (thường bỏ qua nhưng quan trọng)
sudo nmap -sU --top-ports 100 -oA scans/udp <IP>
```

---

## 🔍 Scan Types

|Lệnh|Mô tả|
|---|---|
|`nmap -sS`|SYN Scan (stealth, cần root) — **mặc định dùng**|
|`nmap -sT`|TCP Connect Scan (không cần root)|
|`nmap -sU`|UDP Scan|
|`nmap -sN`|NULL Scan (bypass một số firewall)|
|`nmap -sF`|FIN Scan|
|`nmap -sX`|Xmas Scan|
|`nmap -sA`|ACK Scan (detect firewall rules)|
|`nmap -sW`|Window Scan|
|`nmap -sM`|Maimon Scan|
|`nmap -sn`|Ping Scan — host discovery only, no port scan|
|`nmap -Pn`|**Skip ping** — luôn dùng khi host block ICMP|

---

## 🎯 Port Specification

```bash
nmap -p 80                  # single port
nmap -p 22,80,443           # multiple ports
nmap -p 1-1000              # range
nmap -p-                    # ALL 65535 ports ← IppSec style
nmap --top-ports 100        # top 100 most common
nmap --top-ports 1000       # top 1000 (default khi không chỉ định)
nmap -F                     # Fast scan — top 100 ports
```

---

## 🧠 Service & OS Detection

```bash
nmap -sV                    # version detection
nmap -sV --version-intensity 9    # aggressive version (chậm hơn nhưng chính xác hơn)
nmap -O                     # OS detection (cần root)
nmap -A                     # Aggressive: -O -sV -sC --traceroute
nmap -sCV                   # = -sC -sV ← IppSec dùng thường xuyên
```

---

## 📜 Default Scripts & NSE

```bash
nmap -sC                    # Default scripts
nmap --script=default       # Giống -sC
nmap --script=vuln          # Scan lỗ hổng phổ biến
nmap --script=auth          # Thử auth mặc định / anonymous login
nmap --script=discovery     # Thu thập thêm thông tin
nmap --script=safe          # Scripts an toàn, không gây crash
nmap --script=intrusive     # Có thể gây crash — dùng cẩn thận
nmap --script=exploit       # Exploit scripts

# Script cụ thể
nmap --script=http-title -p 80,443,8080    # Lấy title web
nmap --script=smb-vuln-ms17-010            # EternalBlue check
nmap --script=ftp-anon -p 21              # FTP anonymous login
nmap --script=ssh-auth-methods -p 22      # SSH auth methods

# Truyền args vào script
nmap --script=http-brute --script-args userdb=users.txt,passdb=pass.txt -p 80
```

---

## 🚀 Tốc độ & Timing

|Flag|Tên|Mô tả|
|---|---|---|
|`-T0`|Paranoid|Cực chậm, bypass IDS|
|`-T1`|Sneaky|Chậm|
|`-T2`|Polite|Giảm tải network|
|`-T3`|Normal|Mặc định|
|`-T4`|Aggressive|**IppSec dùng thường**|
|`-T5`|Insane|Rất nhanh, có thể miss port|

```bash
# IppSec style — nhanh + ổn định
nmap -p- --min-rate 10000 -T4 <IP>

# Kiểm soát thủ công
nmap --min-rate 5000 --max-retries 1 -p- <IP>
nmap --max-rate 100    # Giới hạn packet/s
nmap --scan-delay 1s   # Delay giữa các probe
```

---

## 💾 Output Formats

```bash
nmap -oN output.txt     # Normal (human readable)
nmap -oX output.xml     # XML
nmap -oG output.gnmap   # Grepable ← dùng để grep nhanh
nmap -oA scans/result   # ALL formats cùng lúc ← IppSec luôn dùng
nmap -oS output.txt     # Script kiddie (haha)

# Grep từ grepable output
grep "open" output.gnmap
grep "80/open" output.gnmap | awk '{print $2}'
```

---

## 🌐 Host Discovery

```bash
# Không scan port, chỉ discover host
nmap -sn 192.168.1.0/24

# Các phương pháp ping
nmap -PS22,80,443    # TCP SYN ping
nmap -PA80           # TCP ACK ping
nmap -PU53           # UDP ping
nmap -PE             # ICMP echo
nmap -PP             # ICMP timestamp
nmap -PM             # ICMP netmask

# Liệt kê targets (không scan)
nmap -sL 192.168.1.0/24

# Bỏ qua host discovery (treat all as online)
nmap -Pn <IP>        # ← QUAN TRỌNG khi host block ping
```

---

## 🔒 Firewall / IDS Evasion

```bash
# Fragment packets
nmap -f <IP>
nmap -ff <IP>           # Fragment thêm (8 bytes)
nmap --mtu 24 <IP>      # Custom MTU (phải chia hết 8)

# Decoy scan — giả mạo IP nguồn
nmap -D RND:10 <IP>                         # 10 IP ngẫu nhiên
nmap -D 10.0.0.1,10.0.0.2,ME <IP>          # IP cụ thể + IP thật

# Source port spoofing
nmap --source-port 53 <IP>      # Giả là DNS (thường bypass firewall)
nmap -g 80 <IP>                 # Giống --source-port

# Spoof MAC
nmap --spoof-mac Apple <IP>
nmap --spoof-mac 0 <IP>         # Random MAC

# Idle/Zombie scan (hoàn toàn ẩn danh)
nmap -sI <zombie_IP> <target>

# Slow scan
nmap -T1 --scan-delay 5s <IP>

# Bad checksum (bypass một số firewall)
nmap --badsum <IP>
```

---

## 🎯 Các Scenario Thực Chiến

### Web Enumeration

```bash
nmap -p 80,443,8080,8443,8888 -sCV --script=http-title,http-headers,http-methods <IP>
```

### SMB / Windows

```bash
nmap -p 139,445 -sCV --script=smb-enum-shares,smb-enum-users,smb-vuln-ms17-010 <IP>
nmap -p 139,445 --script=smb-vuln-* <IP>
```

### FTP

```bash
nmap -p 21 -sCV --script=ftp-anon,ftp-bounce,ftp-syst <IP>
```

### SSH

```bash
nmap -p 22 -sCV --script=ssh-auth-methods,ssh-hostkey,ssh2-enum-algos <IP>
```

### DNS

```bash
nmap -p 53 -sU -sCV --script=dns-zone-transfer,dns-brute <IP>
```

### SNMP

```bash
nmap -p 161 -sU --script=snmp-info,snmp-sysdescr,snmp-brute <IP>
```

### Database

```bash
nmap -p 1433 --script=ms-sql-info,ms-sql-empty-password <IP>   # MSSQL
nmap -p 3306 --script=mysql-info,mysql-empty-password <IP>      # MySQL
nmap -p 5432 --script=pgsql-brute <IP>                          # PostgreSQL
nmap -p 1521 --script=oracle-tns-version <IP>                   # Oracle
nmap -p 6379 --script=redis-info <IP>                           # Redis
```

### RDP / WinRM

```bash
nmap -p 3389 --script=rdp-enum-encryption,rdp-vuln-ms12-020 <IP>
nmap -p 5985,5986 -sCV <IP>   # WinRM
```

### LDAP / Active Directory

```bash
nmap -p 389,636,3268,3269 -sCV --script=ldap-rootdse,ldap-search <IP>
nmap -p 88 -sCV <IP>   # Kerberos
```

---

## 🗃️ IppSec Full Workflow Template

```bash
TARGET=10.10.10.X
mkdir -p scans

# 1. Full port scan
nmap -p- --min-rate 10000 -oA scans/allports $TARGET

# 2. Extract open ports
PORTS=$(grep "open" scans/allports.gnmap | grep -oP '\d+/open' | cut -d/ -f1 | tr '\n' ',')
echo $PORTS

# 3. Deep scan trên open ports
nmap -p $PORTS -sCV -oA scans/targeted $TARGET

# 4. UDP top ports
sudo nmap -sU --top-ports 20 -oA scans/udp $TARGET

# 5. Vuln scan (tùy chọn)
nmap -p $PORTS --script=vuln -oA scans/vuln $TARGET
```

---

## 🛠️ Useful Options

```bash
-v / -vv / -vvv         # Verbose (IppSec hay dùng -v)
--reason                # Hiện lý do port open/closed
--open                  # Chỉ hiện port open
--packet-trace          # Debug packets
--iflist                # Liệt kê interfaces
--version-trace         # Debug version detection
-n                      # Không resolve DNS ← tăng tốc
-R                      # Luôn resolve DNS
--dns-servers 8.8.8.8   # Dùng DNS server cụ thể
--traceroute            # Traceroute
-6                      # IPv6 scan
--privileged            # Giả sử có root privileges
--unprivileged          # Giả sử không có root
--resume scan.gnmap     # Tiếp tục scan bị gián đoạn
```

---

## 🧩 Script Categories

|Category|Mô tả|
|---|---|
|`auth`|Bypass hoặc brute-force authentication|
|`broadcast`|Discover hosts qua broadcast|
|`brute`|Brute-force credentials|
|`default`|Scripts chạy với -sC|
|`discovery`|Thu thập thông tin|
|`dos`|Denial of Service tests|
|`exploit`|Khai thác lỗ hổng|
|`external`|Dùng external resources|
|`fuzzer`|Fuzzing|
|`intrusive`|Có thể gây ảnh hưởng target|
|`malware`|Detect malware/backdoors|
|`safe`|Không gây hại|
|`version`|Version detection|
|`vuln`|Vulnerability checks|

---

## 📂 NSE Script Paths

```bash
# Tìm scripts
locate *.nse
find /usr/share/nmap/scripts/ -name "*.nse" | grep smb
ls /usr/share/nmap/scripts/ | grep http

# Xem script info
nmap --script-help smb-vuln-ms17-010
nmap --script-help "http-*"
```

---

## 🔗 Kết hợp với Tools khác

```bash
# Scan từ danh sách IP
nmap -iL hosts.txt -sCV -oA scans/from-list

# Loại trừ IP
nmap 192.168.1.0/24 --exclude 192.168.1.1

# Pipe ports vào Gobuster / Nikto
nmap -p 80,443 <IP> -oG - | grep "open" | awk '{print $2}' | \
  xargs -I{} nikto -host {}

# Export XML → HTML report
xsltproc scans/result.xml -o report.html
```

---

## 📌 Tips từ IppSec & Community

- **Luôn dùng `-oA`** để lưu cả 3 format, tiếc công scan lại rất mất thời gian.
- **`-p-` trước, chi tiết sau** — full port scan trước, rồi mới targeted.
- **`-Pn` khi không ping được** — nhiều HTB/OSCP box block ICMP.
- **`--min-rate 10000`** cho CTF/lab, thực tế dùng thấp hơn.
- **NSE `vuln`** đôi khi tạo noise lớn, dùng sau khi đã enumerate xong.
- **UDP thường bị bỏ qua** nhưng hay có SNMP, DNS, TFTP ẩn.
- **`--script-help`** là bạn — đọc trước khi chạy script lạ.
- **`-sCV` = `-sC -sV`** — shorthand IppSec dùng gần như mọi lúc.

---

_Compiled for pentesters — HTB / OSCP / real-world engagements_  
_Inspired by IppSec's methodology @ youtube.com/c/ippsec_