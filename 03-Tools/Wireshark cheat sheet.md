# 🦈 Wireshark Cheat Sheet — Pentest & CTF

> Tổng hợp các kỹ thuật Wireshark được dùng bởi IppSec, John Hammond, và các hacker uy tín trong pentest / CTF.

---

## 📌 Mở file PCAP nhanh

```bash
wireshark capture.pcap
tshark -r capture.pcap          # CLI alternative
```

---

## 🔍 Display Filters (Quan trọng nhất)

### Theo Protocol

```
http
ftp
dns
smb
smb2
ssh
telnet
ssl || tls
icmp
arp
kerberos
ldap
rdp
```

### Theo IP / Port

```
ip.addr == 10.10.10.1
ip.src == 10.10.10.1
ip.dst == 10.10.10.1
tcp.port == 4444
udp.port == 53
ip.addr == 10.10.10.0/24
```

### Lọc kết hợp

```
ip.src == 10.10.10.1 && tcp.port == 80
http && ip.dst == 10.10.10.5
!(arp || dns || icmp)                   # Loại bỏ noise
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN scan
```

---

## 🌐 HTTP / Web Exploitation

```
http.request                            # Tất cả HTTP requests
http.response                           # Tất cả HTTP responses
http.request.method == "POST"           # POST requests (login, upload)
http.request.method == "GET"
http.response.code == 200
http.response.code == 401               # Auth required
http.response.code == 500               # Server error
http.host == "target.com"
http.request.uri contains "admin"
http.request.uri contains "passwd"
http.cookie contains "session"
http.authorization                      # Basic Auth credentials
```

### Extract HTTP Objects (File Download/Upload)

```
File → Export Objects → HTTP
```

---

## 🔐 Credentials Hunting

### FTP (Cleartext)

```
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp
```

### Telnet (Cleartext)

```
telnet
Follow → TCP Stream  → xem toàn bộ session
```

### HTTP Basic Auth

```
http.authorization
```

Decode Base64 trong phần `Authorization: Basic <base64>`

### SMTP / Email

```
smtp
smtp.req.command == "AUTH"
```

### Kerberos (AD Pentest)

```
kerberos
kerberos.CNameString                    # Username
kerberos.as_req_element                 # AS-REQ (Pre-auth)
```

---

## 🧲 Follow Stream (Cực kỳ quan trọng trong CTF)

|Shortcut|Action|
|---|---|
|Chuột phải → Follow → TCP Stream|Xem toàn bộ TCP session|
|Chuột phải → Follow → UDP Stream|UDP session|
|Chuột phải → Follow → HTTP Stream|HTTP request + response|
|Chuột phải → Follow → TLS Stream|TLS (cần key)|

> **Tip IppSec:** Luôn Follow TCP Stream khi thấy reverse shell hoặc C2 traffic — bạn sẽ thấy toàn bộ lệnh được thực thi.

---

## 🕵️ Network Scanning Detection

```
tcp.flags.syn == 1 && tcp.flags.ack == 0     # SYN scan (Nmap -sS)
tcp.flags == 0x001                            # FIN scan
tcp.flags == 0x029                            # Xmas scan
icmp.type == 8                                # Ping sweep
udp && length == 0                            # UDP scan
```

---

## 📡 DNS Analysis

```
dns
dns.qry.name contains "evil"
dns.qry.type == 1                        # A records
dns.qry.type == 28                       # AAAA records
dns.qry.type == 16                       # TXT records (DNS exfil)
dns.resp.len > 100                       # Suspiciously large DNS response
```

### DNS Exfiltration Detection

```
dns.qry.name matches ".*[a-f0-9]{20,}.*"   # Base32/hex subdomains
dns && frame.len > 200                       # Oversized DNS packets
```

---

## 🗂️ SMB / Windows — AD Pentest

```
smb2
smb2.cmd == 5                           # SMB2 Create (file access)
smb2.filename contains "password"
smb2.filename contains ".ps1"
smb                                     # SMBv1 (EternalBlue target)
ntlmssp                                 # NTLM auth
ntlmssp.auth.username                   # Username trong NTLM
```

### Extract SMB Files

```
File → Export Objects → SMB
```

---

## 🔓 TLS / SSL — Decrypt Traffic

### Dùng Private Key

```
Edit → Preferences → Protocols → TLS
→ RSA Keys → thêm: IP, Port, Protocol, Key file (.pem/.key)
```

### Dùng SSLKEYLOGFILE (Firefox/Chrome)

```bash
export SSLKEYLOGFILE=~/ssl-keys.log
firefox &
```

Sau đó trong Wireshark:

```
Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename
```

---

## 🐚 Reverse Shell & C2 Traffic

```
tcp.port == 4444                         # Metasploit default
tcp.port == 1234 || tcp.port == 9001
tcp contains "bash"
tcp contains "cmd.exe"
tcp contains "powershell"
frame contains "whoami"
frame contains "id"
```

> **Tip:** Dùng `Statistics → Conversations` để xem kết nối nào tồn tại lâu bất thường → dấu hiệu C2 beaconing.

---

## 📊 Statistics (Phân tích nhanh)

|Menu|Công dụng|
|---|---|
|`Statistics → Protocol Hierarchy`|Tổng quan protocols trong PCAP|
|`Statistics → Conversations`|Top kết nối theo bytes/packets|
|`Statistics → Endpoints`|Tất cả IPs liên quan|
|`Statistics → IO Graphs`|Visualize traffic theo thời gian|
|`Statistics → HTTP → Requests`|Tất cả HTTP requests|
|`Analyze → Expert Information`|Lỗi, cảnh báo tự động|

---

## 🛠️ tshark — CLI Wireshark

```bash
# List interfaces
tshark -D

# Capture live
tshark -i eth0 -w output.pcap

# Read và lọc
tshark -r capture.pcap -Y "http.request"

# Xuất fields cụ thể
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e http.request.uri

# Theo dõi TCP stream
tshark -r capture.pcap -q -z follow,tcp,ascii,0

# Thống kê protocols
tshark -r capture.pcap -q -z io,phs

# Export HTTP objects
tshark -r capture.pcap --export-objects http,/tmp/output/

# Lọc và đếm
tshark -r capture.pcap -Y "dns" | wc -l
```

---

## 🚩 CTF Tips — IppSec Style

### 1. Workflow khi nhận PCAP trong CTF

```
1. Statistics → Protocol Hierarchy        # Có gì trong file?
2. Statistics → Conversations             # Ai nói chuyện với ai?
3. File → Export Objects → HTTP/SMB/FTP   # File nào được transfer?
4. Filter by protocol quan trọng          # http, ftp, dns, smb2
5. Follow TCP/UDP Stream trên kết nối lạ
6. Tìm strings: frame contains "flag"
```

### 2. Tìm Flag trực tiếp

```
frame contains "HTB{"
frame contains "CTF{"
frame contains "flag{"
frame contains "FLAG"
tcp contains "flag"
```

### 3. Data Exfiltration qua DNS

```
dns.qry.name matches ".*\.attacker\.com"
```

Decode subdomain từ base64/hex để lấy dữ liệu.

### 4. Steganography trong PCAP

```bash
# Extract tất cả files
binwalk -e capture.pcap
foremost -i capture.pcap -o output/
```

### 5. ICMP Data Exfil

```
icmp
# Xem Data field trong ICMP Echo Request
```

---

## 🔧 Keyboard Shortcuts

|Shortcut|Action|
|---|---|
|`Ctrl+F`|Find packet|
|`Ctrl+G`|Go to packet number|
|`Ctrl+E`|Follow stream|
|`Ctrl+Shift+X`|Export objects|
|`Ctrl+Alt+Shift+T`|Protocol hierarchy|
|`Space`|Next packet|
|`Backspace`|Previous packet|

---

## 🏷️ Coloring Rules (Mặc định hữu ích)

|Màu|Ý nghĩa|
|---|---|
|🔴 Đỏ|Lỗi TCP (RST, checksum error)|
|🟡 Vàng|Cảnh báo (out-of-order, retransmission)|
|🟢 Xanh lá|HTTP traffic|
|🔵 Xanh dương|DNS, TCP|
|⚫ Đen trên đỏ|Lỗi nghiêm trọng|

---

## 📚 References

- [IppSec YouTube](https://www.youtube.com/@ippsec) — HTB writeups với Wireshark analysis
- [Wireshark Docs](https://www.wireshark.org/docs/)
- [PacketLife Cheat Sheet](https://packetlife.net/media/library/13/Wireshark_Display_Filters.pdf)
- [Hack The Box PCAP Challenges](https://app.hackthebox.com/)

---

_Last updated: 2026 | Dành cho pentesters & CTF players_