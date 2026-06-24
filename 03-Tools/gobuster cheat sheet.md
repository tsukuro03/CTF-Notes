# 🔍 Gobuster Cheatsheet — Pentest Edition

> Tổng hợp lệnh thực chiến từ IppSec, HackTheBox, TryHackMe & cộng đồng pentest uy tín

---

## 📦 Cài đặt

```bash
# Kali / Debian
sudo apt install gobuster

# Từ source (Go)
go install github.com/OJ/gobuster/v3@latest

# Kiểm tra version
gobuster version
```

---

## 🧭 Các Mode chính

|Mode|Mô tả|
|---|---|
|`dir`|Brute-force thư mục / file|
|`dns`|Brute-force subdomain|
|`vhost`|Brute-force Virtual Host|
|`fuzz`|Fuzzing tham số tùy chỉnh|
|`s3`|Enumerate AWS S3 buckets|
|`gcs`|Enumerate Google Cloud buckets|
|`tftp`|Enumerate TFTP server|

---

## 📁 DIR Mode — Brute-force Thư mục / File

### Cơ bản

```bash
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
```

### Phổ biến nhất (IppSec style)

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,html,txt,bak \
  -t 50 \
  -o gobuster_dir.txt
```

### Với Authentication (Basic Auth)

```bash
gobuster dir \
  -u http://10.10.10.10/admin \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt \
  -U admin -P password123 \
  -t 40
```

### Với Cookie / Session

```bash
gobuster dir \
  -u http://10.10.10.10/dashboard \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt \
  -c "PHPSESSID=abc123def456; security=low" \
  -t 50
```

### Với Custom Header (JWT, API token)

```bash
gobuster dir \
  -u http://10.10.10.10/api \
  -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -t 40
```

### Bypass 403 / Filter status code

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
  -b 404,403,500 \
  -t 50
```

### Chỉ lấy status code cụ thể

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  --status-codes 200,204,301,302,307 \
  -t 50
```

### Với HTTPS (ignore cert)

```bash
gobuster dir \
  -u https://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -k \
  -t 50
```

### Proxy qua Burp Suite

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  --proxy http://127.0.0.1:8080 \
  -t 30
```

### Enumerate file extensions cụ thể (API endpoints)

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt \
  -x php,aspx,jsp,py,txt,conf,config,bak,backup,old,log,xml,json \
  -t 60
```

### Thêm trailing slash (tìm thư mục ẩn)

```bash
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  --add-slash \
  -t 40
```

---

## 🌐 DNS Mode — Brute-force Subdomain
-Hãy tưởng tượng internet như một cuốn danh bạ điện thoại khổng lồ. Máy tính chỉ hiểu địa chỉ IP như 192.168.1.1, còn con người thì nhớ tên như google.com. DNS là người phiên dịch ở giữa.

-Khi bạn gõ google.com vào trình duyệt, máy tính hỏi DNS server: "google.com là IP nào vậy?". DNS server trả lời: "142.250.x.x". Sau đó trình duyệt mới biết địa chỉ để kết nối tới.

-Vì vậy gobuster dns hoạt động bằng cách hỏi DNS server: "dev.team.thm có tồn tại không?". Nhưng nếu domain giả trong CTF nghĩa là domain đó phải ghi vào file host mới tìm được đường đến, không có DNS server nào biết nó, nên câu trả lời luôn là "không biết" và gobuster không tìm được gì.
### Cơ bản

```bash
gobuster dns \
  --domain example.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Với wildcard bypass + hiển thị IP

```bash
gobuster dns \
  --domain example.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -t 50 \
  --show-ips
```

### Dùng resolver tùy chỉnh (dùng DNS nội bộ HTB/THM)

```bash
gobuster dns \
  --domain example.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -r 10.10.10.10:53 \
  -t 30
```

### Subdomain với wildcard (bỏ qua kết quả wildcard)

```bash
gobuster dns \
  --domain example.com \
  -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt \
  --wildcard \
  -t 50
```

---

## 🖥️ VHOST Mode — Virtual Host Enumeration
-Bây giờ tưởng tượng một tòa nhà chung cư. Địa chỉ tòa nhà chỉ có một, nhưng bên trong có nhiều căn hộ khác nhau. Người đưa thư cần biết thêm số phòng để giao đúng chỗ.

-Web server cũng vậy. Một IP có thể chạy nhiều website cùng lúc. Khi trình duyệt gửi request, nó luôn kèm theo một header tên là Host để nói rõ muốn vào website nào.

-Ví dụ cụ thể, cùng một IP nhưng:

- Host: team.thm → trả về trang chủ
- Host: dev.team.thm → trả về trang dev
- Host: admin.team.thm → trả về trang admin

Web server đọc cái Host header đó rồi quyết định trả về nội dung nào.
### Cơ bản (quan trọng trong CTF / real pentest)

```bash
gobuster vhost \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain
```

### Với domain gốc (HTB style)

```bash
gobuster vhost \
  -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -H "Host: FUZZ.example.htb" \
  --append-domain \
  -t 50
```

### Lọc theo response size (tránh false positive)

```bash
gobuster vhost \
  -u http://example.htb \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --exclude-length 301 \
  --append-domain \
  -t 40
```

---

## 🔀 FUZZ Mode — Fuzzing tham số

### Fuzz parameter trong URL

```bash
gobuster fuzz \
  -u "http://10.10.10.10/index.php?FUZZ=test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -b 404
```

### Fuzz POST body

```bash
gobuster fuzz \
  -u http://10.10.10.10/login \
  -w /usr/share/seclists/Passwords/Common-Credentials/top-passwords-shortlist.txt \
  -m POST \
  -d "username=admin&password=FUZZ" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -b 200
```

---

## ☁️ S3 Mode — AWS Bucket Enumeration

```bash
gobuster s3 \
  -w /usr/share/seclists/Discovery/S3/bucket-names.txt \
  -t 50

# Với prefix tên công ty
gobuster s3 \
  -w /usr/share/seclists/Discovery/S3/bucket-names.txt \
  --prefix "companyname-" \
  -t 30
```

---

## 📝 Wordlists được dùng nhiều nhất

```
# Thư mục & file (SecLists - khuyên dùng nhất)
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-words.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt

# API endpoints
/usr/share/seclists/Discovery/Web-Content/api/objects.txt
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Subdomain / DNS
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
/usr/share/seclists/Discovery/DNS/dns-Jhaddix.txt

# Cài SecLists nếu chưa có
sudo apt install seclists
```

---

## ⚙️ Các flag quan trọng

|Flag|Mô tả|
|---|---|
|`-u`|Target URL|
|`-w`|Wordlist path|
|`-t`|Threads (mặc định 10, tối đa ~100)|
|`-x`|Extension (php,html,txt...)|
|`-o`|Output file|
|`-k`|Bỏ qua SSL verify|
|`-b`|Blacklist status codes (bỏ qua)|
|`--status-codes`|Chỉ hiện status codes này|
|`-c`|Cookie|
|`-H`|Custom header|
|`-U` / `-P`|HTTP Basic Auth|
|`--proxy`|Proxy URL|
|`-r`|Follow redirects|
|`-q`|Quiet mode (chỉ in kết quả)|
|`--delay`|Delay giữa các request (ms)|
|`--timeout`|Timeout mỗi request|
|`--add-slash`|Thêm `/` vào cuối|
|`--no-tls-validation`|Tương tự `-k`|
|`--exclude-length`|Loại bỏ response theo size|
|`-m`|HTTP method (GET, POST, HEAD...)|
|`-d`|POST data|

---

## 💡 Tips & Tricks thực chiến

### 1. Tốc độ tối ưu

```bash
# Bắt đầu với 50 threads, tăng lên 100 nếu target ổn định
-t 50

# Thêm delay nếu gặp rate limiting
--delay 100ms
```

### 2. Pipeline với tee (vừa xem vừa lưu)

```bash
gobuster dir -u http://10.10.10.10 -w /wordlist.txt -t 50 | tee gobuster.txt
```

### 3. Chạy song song nhiều extension

```bash
# Ưu tiên file hay gặp trong CTF
-x php,html,txt,bak,zip,tar.gz,sql,log,config,conf,xml,json,js
```

### 4. Khi gặp 403 Forbidden

```bash
# Thử thêm trailing slash
--add-slash

# Kết hợp với feroxbuster để bypass 403
feroxbuster -u http://10.10.10.10 --dont-filter
```

### 5. IppSec Workflow chuẩn

```bash
# Bước 1: Scan nhanh
gobuster dir -u http://TARGET -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 40

# Bước 2: Scan sâu
gobuster dir -u http://TARGET -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -x php,txt,html -t 50 -o full_scan.txt

# Bước 3: Scan từng thư mục tìm được
gobuster dir -u http://TARGET/admin -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -x php,bak -t 40
```

### 6. Virtual Host (quan trọng trong HTB)

```bash
# Thêm vào /etc/hosts trước
echo "10.10.10.10 example.htb" >> /etc/hosts

# Rồi scan vhost
gobuster vhost -u http://example.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain -t 50
```

### 7. API Enumeration

```bash
gobuster dir \
  -u http://10.10.10.10/api/v1 \
  -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -x json \
  -t 40
```

---

## ⚠️ Lưu ý khi dùng

- **Luôn có phép** trước khi scan — chỉ dùng trên lab (HTB, THM, DVWA...) hoặc môi trường bạn sở hữu
- Giảm threads nếu gặp lỗi connection / rate limit
- Kết hợp với `ffuf` hoặc `feroxbuster` để cross-verify
- Dùng `-o` để lưu output, tránh mất kết quả
- Trên các box HTB: hay gặp vhost ẩn → **luôn chạy vhost scan**

---

## 🔗 Tài nguyên

- [GitHub Gobuster](https://github.com/OJ/gobuster)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [IppSec YouTube](https://www.youtube.com/@ippsec)
- [HackTricks - Gobuster](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/web-content-discovery)