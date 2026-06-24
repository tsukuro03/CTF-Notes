## 📦 Cú pháp cơ bản

```
tcpdump [options] [expression]
```

---

## 🔧 Options phổ biến nhất

|Option|Mô tả|
|---|---|
|`-i eth0`|Chỉ định interface (dùng `-i any` để bắt tất cả)|
|`-n`|Không resolve DNS (nhanh hơn, không bị nhiễu)|
|`-nn`|Không resolve DNS **và** port names|
|`-v` / `-vv` / `-vvv`|Tăng độ chi tiết output|
|`-X`|Hiển thị payload dạng HEX + ASCII|
|`-A`|Hiển thị payload dạng ASCII (tốt cho HTTP)|
|`-e`|Hiển thị Ethernet/MAC header|
|`-c <n>`|Bắt đúng n gói rồi dừng|
|`-s <n>`|Snap length — dùng `-s 0` hoặc `-s 65535` để bắt full packet|
|`-w file.pcap`|Ghi ra file pcap|
|`-r file.pcap`|Đọc từ file pcap|
|`-l`|Line-buffered — dùng khi pipe sang `grep`|
|`-q`|Quiet mode — ít output hơn|
|`-t`|Không in timestamp|
|`-tttt`|In timestamp dạng human-readable|
|`-D`|Liệt kê tất cả interfaces có thể bắt|

---

## 🎯 Bộ lọc (BPF Filters)

### Lọc theo Host

```bash
tcpdump host 10.10.10.1                    # bất kỳ traffic nào với host
tcpdump src host 10.10.10.1               # chỉ traffic từ host
tcpdump dst host 10.10.10.1               # chỉ traffic đến host
tcpdump not host 10.10.10.1               # loại trừ host
```

### Lọc theo Port

```bash
tcpdump port 80                            # port 80 (src hoặc dst)
tcpdump src port 443                       # chỉ từ port 443
tcpdump dst port 22                        # chỉ đến port 22
tcpdump portrange 1-1024                   # dải port
tcpdump not port 22                        # loại trừ SSH
```

### Lọc theo Protocol

```bash
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp
tcpdump ip6
tcpdump proto 17                           # UDP by protocol number
```

### Lọc theo Network

```bash
tcpdump net 10.10.10.0/24
tcpdump src net 192.168.1.0/24
tcpdump dst net 172.16.0.0/12
```

### Toán tử logic

```bash
tcpdump host 10.10.10.1 and port 80
tcpdump host 10.10.10.1 or host 10.10.10.2
tcpdump not icmp
tcpdump (host 10.10.10.1 and port 80) or (host 10.10.10.2 and port 443)
```

---

## 🔍 Lọc TCP Flags (dùng nhiều trong CTF)

```bash
# SYN packets (port scan detection / new connections)
tcpdump 'tcp[tcpflags] == tcp-syn'

# SYN-ACK
tcpdump 'tcp[tcpflags] == (tcp-syn|tcp-ack)'

# RST packets (port closed / firewall resets)
tcpdump 'tcp[tcpflags] & (tcp-rst) != 0'

# FIN packets
tcpdump 'tcp[tcpflags] & (tcp-fin) != 0'

# PSH+ACK (data being sent)
tcpdump 'tcp[tcpflags] & (tcp-push|tcp-ack) != 0'

# URG flag (thường bị dùng trong covert channel)
tcpdump 'tcp[tcpflags] & tcp-urg != 0'
```

---

## 📡 Các lệnh hay dùng trong CTF & Pentest

### Bắt toàn bộ traffic, ghi vào file

```bash
tcpdump -i any -nn -s0 -w capture.pcap
```

### Xem HTTP request/response (plain text)

```bash
tcpdump -i eth0 -A -s0 port 80
```

### Xem DNS queries

```bash
tcpdump -i any -nn port 53
```

### Xem ICMP (ping, traceroute, ICMP tunneling)

```bash
tcpdump -i any -nn icmp
tcpdump -i any -nn icmp and host 10.10.10.1
```

### Theo dõi SSH traffic (không xem được nội dung, nhưng thấy pattern)

```bash
tcpdump -i eth0 -nn port 22
```

### Bắt credential trong cleartext (FTP, Telnet, Basic Auth)

```bash
tcpdump -i eth0 -A -s0 'port ftp or port 23 or port 21'
```

### Phát hiện port scan (SYN flood từ 1 IP)

```bash
tcpdump -i eth0 -nn 'tcp[tcpflags] == tcp-syn' and src host <attacker_ip>
```

### Xem ARP (network discovery, ARP spoofing)

```bash
tcpdump -i eth0 -nn arp
```

### Lọc traffic không phải từ mình (loại bỏ noise)

```bash
tcpdump -i eth0 -nn not src host 10.10.14.5
```

---

## 🕵️ Kỹ thuật của IppSec & Pro Players

### Pattern IppSec hay dùng khi bắt đầu box

```bash
# Bắt full traffic ngay từ đầu, phân tích sau
sudo tcpdump -i tun0 -w /tmp/box_name.pcap -s0

# Xem nhanh có gì đang chạy
sudo tcpdump -i tun0 -nn -v
```

### Bắt và grep realtime (cực hữu ích)

```bash
tcpdump -i eth0 -A -l | grep -i "password\|pass\|user\|login"
tcpdump -i eth0 -A -l | grep -iE "(GET|POST|Host:|Authorization:)"
tcpdump -i eth0 -A -l | grep -i "flag\|HTB\|THM"
```

### Xem content của ICMP data payload (ICMP tunnel detection)

```bash
tcpdump -i any icmp -X
```

### Phân tích pcap sẵn có

```bash
tcpdump -r capture.pcap -nn -A | less
tcpdump -r capture.pcap -nn 'tcp and port 80' -A
tcpdump -r capture.pcap -nn 'not arp and not icmp'
```

### Extract credentials từ pcap

```bash
tcpdump -r capture.pcap -A | grep -i "authorization: basic" | \
  awk '{print $3}' | base64 -d
```

---

## 🧠 BPF Advanced Filters (offset-based)

```bash
# HTTP GET requests
tcpdump 'tcp[20:4] = 0x47455420'          # "GET "

# HTTP POST requests  
tcpdump 'tcp[20:4] = 0x504f5354'          # "POST"

# SSH version string (banner grab detection)
tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482d'  # "SSH-"

# ICMP type 8 (echo request / ping)
tcpdump 'icmp[icmptype] = icmp-echo'

# ICMP với data lớn hơn bình thường (có thể là tunnel)
tcpdump 'icmp and ip[2:2] > 100'

# TCP packets với data (không phải pure ACK)
tcpdump 'tcp and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# DNS response (flag QR=1)
tcpdump 'udp port 53 and udp[10] & 0x80 != 0'
```

---

## 🏁 CTF-Specific Scenarios

### Scenario 1: Tìm credentials trong traffic

```bash
# Chạy trước khi submit form / login
tcpdump -i any -A -nn port 80 or port 8080 or port 443 | \
  grep -iE "(pass|user|login|token|key|secret)"
```

### Scenario 2: Phát hiện reverse shell callback

```bash
# Monitor outbound connections bất thường
tcpdump -i any -nn 'not port 80 and not port 443 and not port 22 and tcp[tcpflags] == tcp-syn'
```

### Scenario 3: SNMP community string leak

```bash
tcpdump -i any -nn -A udp port 161
```

### Scenario 4: Kerberos / AD traffic (Windows boxes)

```bash
tcpdump -i any -nn port 88                 # Kerberos
tcpdump -i any -nn port 389 or port 636    # LDAP / LDAPS
tcpdump -i any -nn port 445                # SMB
```

### Scenario 5: Exfiltration qua DNS

```bash
# DNS queries bất thường (tên domain rất dài)
tcpdump -i any -nn -A udp port 53 | grep -v "\.arpa"
```

---

## ⚡ One-liners hay nhất

```bash
# Bắt và xem ngay top talkers
tcpdump -i eth0 -nn -q | awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn | head

# Live HTTP hosts đang được truy cập
tcpdump -i any -A -nn port 80 2>/dev/null | grep "^Host:"

# Xem toàn bộ TCP streams ngắn gọn
tcpdump -i eth0 -nn -q tcp

# Bắt tất cả ngoại trừ traffic của bạn
tcpdump -i eth0 -nn not src and dst host <your_ip>

# Monitor VPN interface (HTB/THM)
tcpdump -i tun0 -nn -v

# Tìm file transfer (FTP data)
tcpdump -i any -nn -A 'tcp port 20 or tcp port 21'
```

---

## 📁 Làm việc với PCAP

```bash
# Merge nhiều pcap
mergecap -w merged.pcap file1.pcap file2.pcap

# Đọc pcap với filter
tcpdump -r capture.pcap -nn 'host 10.10.10.1 and port 80' -A

# Convert sang text để grep
tcpdump -r capture.pcap -A > capture.txt
strings capture.pcap | grep -i "flag\|HTB{"

# Xem statistics
tcpdump -r capture.pcap -nn -q | wc -l
```

---

## 🔐 Cần quyền root

```bash
# Chạy với sudo
sudo tcpdump ...

# Hoặc set capability (không cần sudo mỗi lần)
sudo setcap cap_net_raw,cap_net_admin=eip $(which tcpdump)
```

---

## 🆚 So sánh nhanh với Wireshark

|Tình huống|Dùng|
|---|---|
|Remote server / no GUI|`tcpdump`|
|Capture rồi phân tích sau|`tcpdump -w` → Wireshark|
|Quick grep / pipeline|`tcpdump` + grep/awk|
|Protocol dissection phức tạp|Wireshark|
|CTF: tìm nhanh credential|`tcpdump -A`|
|CTF: phân tích pcap cho sẵn|Wireshark / `tcpdump -r`|

---

## 📚 Tài liệu tham khảo

- `man tcpdump` — tài liệu gốc đầy đủ nhất
- [https://www.tcpdump.org/manpages/tcpdump.1.html](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [https://danielmiessler.com/study/tcpdump/](https://danielmiessler.com/study/tcpdump/) — Daniel Miessler's guide
- IppSec YouTube — xem cách ông ấy dùng tcpdump kết hợp Wireshark trong các HTB writeups

---

_Cheat sheet này tổng hợp từ docs chính thức của tcpdump, các writeups của IppSec, 0xdf, và kinh nghiệm thực tế trong CTF (HackTheBox, TryHackMe, PicoCTF)._