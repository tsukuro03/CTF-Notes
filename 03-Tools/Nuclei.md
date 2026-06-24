# 🔬 Nuclei – Cheatsheet Đầy Đủ (CTF & CERT)

> **Nuclei** là công cụ quét lỗ hổng bảo mật mã nguồn mở, sử dụng các template YAML có thể tùy chỉnh để phát hiện lỗ hổng, misconfiguration, exposed panels, CVE, và nhiều hơn nữa.  
> GitHub: https://github.com/projectdiscovery/nuclei

---

## 📦 Cài Đặt

```bash
# Cài qua Go
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Cài qua apt (Kali Linux)
sudo apt install nuclei

# Cập nhật templates
nuclei -update-templates

# Cập nhật nuclei binary
nuclei -update
```

---

## 🗂️ Mục Lục

1. [[#1. Cú Pháp Cơ Bản]]
2. [[#2. Flags - Target (Mục tiêu)]]
3. [[#3. Flags - Templates]]
4. [[#4. Flags - Filtering (Lọc)]]
5. [[#5. Flags - Output (Đầu ra)]]
6. [[#6. Flags - Rate Limiting & Performance]]
7. [[#7. Flags - Authentication & Headers]]
8. [[#8. Flags - Network & Proxy]]
9. [[#9. Flags - Reporting]]
10. [[#10. Flags - Debugging & Verbose]]
11. [[#11. Lệnh Thường Dùng Trong CTF]]
12. [[#12. Lệnh Thường Dùng Trong Thi CERT / PenTest]]
13. [[#13. Tổng Hợp Flags Nhanh]]

---

## 1. Cú Pháp Cơ Bản

```bash
nuclei -u <URL>                         # Quét 1 URL
nuclei -l <file.txt>                    # Quét danh sách URL từ file
nuclei -u <URL> -t <template/path>      # Quét với template cụ thể
```

---

## 2. Flags - Target (Mục tiêu)

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-u <url>`|`-target`|Chỉ định **một URL mục tiêu** để quét|
|`-l <file>`|`-list`|Đọc danh sách mục tiêu từ **file văn bản** (mỗi dòng 1 URL)|
|`-resume <file>`||**Tiếp tục** quét từ checkpoint đã lưu (dùng khi bị gián đoạn)|
|`-sa`|`-scan-all-ips`|Quét **tất cả IP** được resolve từ hostname (hữu ích khi 1 domain có nhiều IP)|
|`-iv`|`-include-versions`|In thêm thông tin phiên bản vào output|

**Ví dụ:**

```bash
nuclei -u https://target.com
nuclei -l urls.txt
nuclei -u https://target.com -sa
```

---

## 3. Flags - Templates

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-t <path>`|`-templates`|Chỉ định **đường dẫn template** (file .yaml hoặc thư mục)|
|`-tl`|`-template-list`|**Liệt kê** tất cả template hiện có|
|`-nt`|`-new-templates`|Chỉ chạy các template **mới được thêm** vào lần cập nhật gần nhất|
|`-ntv <version>`|`-new-templates-version`|Chạy template mới từ **phiên bản** được chỉ định|
|`-tu <url>`|`-template-url`|Tải và chạy template từ **URL** (GitHub raw link, v.v.)|
|`-w <path>`|`-workflows`|Chạy **workflow** thay vì template đơn lẻ|
|`-wl`|`-workflow-list`|Liệt kê tất cả workflow có sẵn|
|`-validate`||**Kiểm tra tính hợp lệ** của template YAML trước khi chạy|
|`-update-templates`||**Cập nhật** toàn bộ template lên phiên bản mới nhất|
|`-eti <path>`|`-exclude-templates`|**Loại trừ** template/thư mục cụ thể khỏi lần quét|
|`-include-templates <path>`||**Thêm** template bổ sung vào lần quét|

**Ví dụ:**

```bash
nuclei -u https://target.com -t cves/
nuclei -u https://target.com -t cves/2021/CVE-2021-44228.yaml
nuclei -u https://target.com -t exposures/ -t vulnerabilities/
nuclei -t my-template.yaml -validate
nuclei -u https://target.com -nt
```

---

## 4. Flags - Filtering (Lọc)

### Lọc theo Severity (Mức độ nghiêm trọng)

|Flag|Mô tả|
|---|---|
|`-s <severity>`|Chỉ chạy template có **mức độ nghiêm trọng** nhất định|
|`-es <severity>`|**Loại trừ** template theo mức độ nghiêm trọng|

Các giá trị severity: `info`, `low`, `medium`, `high`, `critical`, `unknown`

```bash
nuclei -u https://target.com -s critical,high
nuclei -u https://target.com -es info,low
```

### Lọc theo Tags

|Flag|Mô tả|
|---|---|
|`-tags <tag>`|Chỉ chạy template có **tag** khớp|
|`-etags <tag>`|**Loại trừ** template theo tag|
|`-itags <tag>`|Chạy template dù bị **exclude by default** nếu có tag khớp|

```bash
nuclei -u https://target.com -tags cve
nuclei -u https://target.com -tags lfi,sqli,xss
nuclei -u https://target.com -etags intrusive
nuclei -u https://target.com -tags oast
```

### Lọc theo Author / ID

|Flag|Mô tả|
|---|---|
|`-author <name>`|Chỉ chạy template của **tác giả** được chỉ định|
|`-id <id>`|Chạy template theo **ID** cụ thể|
|`-eid <id>`|**Loại trừ** template theo ID cụ thể|

```bash
nuclei -u https://target.com -id CVE-2021-44228
nuclei -u https://target.com -author pdteam
```

### Lọc theo Protocol / Type

|Flag|Mô tả|
|---|---|
|`-type <type>`|Lọc theo **loại giao thức** (http, dns, tcp, ssl, ...)|

```bash
nuclei -u https://target.com -type http
nuclei -u https://target.com -type ssl
```

---

## 5. Flags - Output (Đầu ra)

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-o <file>`|`-output`|Lưu kết quả ra **file** (định dạng text)|
|`-json`|`-j`|Xuất kết quả dạng **JSON** (mỗi dòng 1 JSON object)|
|`-jsonl`||Xuất dạng **JSON Lines** (NDJSON)|
|`-sarif-export <file>`||Xuất kết quả dạng **SARIF** (dùng trong CI/CD, GitHub Code Scanning)|
|`-markdown-export <dir>`||Xuất kết quả dạng **Markdown** vào thư mục|
|`-nc`|`-no-color`|Tắt **màu sắc** trong terminal output|
|`-silent`||Chỉ in **kết quả tìm thấy**, không in thông tin khác|
|`-v`|`-verbose`|In thêm thông tin **verbose**|
|`-vv`||In **rất chi tiết** (bao gồm template không match)|
|`-debug`||In toàn bộ **request và response** ra terminal|
|`-debug-req`||In **request** gửi đi|
|`-debug-resp`||In **response** nhận về|
|`-irr`|`-include-rr`|Bao gồm **request/response** trong file output JSON|
|`-nm`|`-no-meta`|Không in **metadata** của template vào output|
|`-nts`|`-no-timestamp`|Không in **timestamp** vào output|
|`-or`|`-omit-raw`|Không in raw request/response trong JSON output|
|`-stats`||Hiển thị **thống kê** quét realtime|
|`-stats-interval <n>`||Cập nhật stats mỗi **n giây**|

**Ví dụ:**

```bash
nuclei -u https://target.com -o results.txt
nuclei -u https://target.com -json -o results.json
nuclei -l urls.txt -json -irr -o full-results.json
nuclei -u https://target.com -silent
nuclei -u https://target.com -stats -stats-interval 5
```

---

## 6. Flags - Rate Limiting & Performance

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-c <n>`|`-concurrency`|Số **goroutine** chạy song song (default: 25)|
|`-bs <n>`|`-bulk-size`|Số template xử lý **mỗi host** cùng lúc (default: 25)|
|`-rl <n>`|`-rate-limit`|Giới hạn **số request/giây** gửi đi|
|`-rlm <n>`|`-rate-limit-minute`|Giới hạn **số request/phút**|
|`-timeout <n>`||**Timeout** (giây) cho mỗi request (default: 5)|
|`-retries <n>`||Số lần **thử lại** khi request thất bại (default: 1)|
|`-mhe <n>`|`-max-host-error`|Số lỗi tối đa trên 1 host trước khi **bỏ qua** host đó|
|`-stream`||Chạy ở **streaming mode** – gửi template ngay khi đọc xong (không buffer)|
|`-ss <n>`|`-system-resolvers`|Dùng **DNS resolver** hệ thống|
|`-ldp`|`-leave-default-ports`|Giữ nguyên **port mặc định** trong URL (không xóa :80/:443)|

**Ví dụ:**

```bash
nuclei -l urls.txt -c 50 -rl 100
nuclei -u https://target.com -timeout 10 -retries 3
nuclei -l urls.txt -c 100 -bs 50 -rl 500
```

---

## 7. Flags - Authentication & Headers

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-H <header>`|`-header`|Thêm **HTTP header** tùy chỉnh vào request|
|`-V <var>`|`-var`|Định nghĩa **biến** tùy chỉnh (dùng trong template)|
|`-auth`||Bật **tự động quản lý xác thực** (dùng với secret file)|
|`-sf <file>`|`-secret-file`|Đường dẫn đến **file chứa thông tin xác thực**|
|`-pa`|`-proxy-auth`|Xác thực **proxy**|

**Ví dụ:**

```bash
# Thêm cookie xác thực
nuclei -u https://target.com -H "Cookie: session=abc123"

# Thêm Authorization header
nuclei -u https://target.com -H "Authorization: Bearer <token>"

# Thêm nhiều header
nuclei -u https://target.com -H "Cookie: session=abc" -H "X-Custom: value"

# Truyền biến cho template
nuclei -u https://target.com -t my-template.yaml -V username=admin -V password=pass123
```

---

## 8. Flags - Network & Proxy

|Flag|Viết tắt|Mô tả|
|---|---|---|
|`-proxy <url>`|`-p`|Sử dụng **HTTP/SOCKS proxy** (vd: http://127.0.0.1:8080)|
|`-proxy-internal`||Chuyển toàn bộ traffic (kể cả internal) qua **proxy**|
|`-systemresolvers`||Dùng **DNS resolver** của hệ thống thay vì resolver mặc định|
|`-resolvers <file>`|`-r`|Chỉ định file chứa **DNS resolver** tùy chỉnh|
|`-no-httpx`||Tắt **httpx probing** cho các input non-HTTP|

**Ví dụ:**

```bash
# Dùng Burp Suite làm proxy để xem traffic
nuclei -u https://target.com -proxy http://127.0.0.1:8080

# Dùng SOCKS5 proxy
nuclei -u https://target.com -proxy socks5://127.0.0.1:1080

# DNS resolver tùy chỉnh
nuclei -u https://target.com -r resolvers.txt
```

---

## 9. Flags - Reporting

|Flag|Mô tả|
|---|---|
|`-rc <file>`|Đường dẫn đến **file cấu hình reporting**|

Nuclei hỗ trợ tích hợp báo cáo với:

- **Jira**: Tự động tạo issue
- **GitHub Issues**: Tạo issue tự động
- **GitLab Issues**: Tạo issue tự động
- **Slack**: Gửi thông báo
- **Microsoft Teams**: Gửi thông báo
- **Email (SMTP)**: Gửi email báo cáo
- **Elasticsearch**: Đẩy dữ liệu vào ES

**Ví dụ file reporting-config.yaml:**

```yaml
# Slack
slack:
  webhook-url: "https://hooks.slack.com/services/xxx"
  username: "Nuclei"
  channel: "#security-alerts"
  severity-filter:
    - high
    - critical
```

```bash
nuclei -l urls.txt -rc reporting-config.yaml
```

---

## 10. Flags - Debugging & Verbose

|Flag|Mô tả|
|---|---|
|`-v`|**Verbose** – In thêm thông tin trong quá trình quét|
|`-vv`|**Very verbose** – In cả template không có kết quả|
|`-debug`|In toàn bộ **raw request và response**|
|`-debug-req`|Chỉ in **raw request** gửi đi|
|`-debug-resp`|Chỉ in **raw response** nhận về|
|`-ep`|**Enable profiling** – Phân tích hiệu năng|
|`-version`|In **phiên bản** Nuclei đang dùng|
|`-health-check`|Kiểm tra **sức khỏe** của Nuclei (connectivity, template, ...)|
|`-ut`|**Update templates** và thoát|
|`-duc`|**Tắt cập nhật tự động** khi khởi động|

**Ví dụ:**

```bash
nuclei -u https://target.com -v
nuclei -u https://target.com -debug 2>&1 | tee debug.log
nuclei -version
nuclei -health-check
```

---

## 11. Lệnh Thường Dùng Trong CTF

### Recon & Discovery

```bash
# Quét tổng quát nhanh - phù hợp với CTF web
nuclei -u http://target.ctf -s medium,high,critical

# Tìm exposed panels (admin, login, dashboard)
nuclei -u http://target.ctf -tags panel

# Tìm file/thư mục nhạy cảm bị lộ
nuclei -u http://target.ctf -tags exposure

# Tìm misconfiguration
nuclei -u http://target.ctf -tags misconfiguration

# Tìm default credentials
nuclei -u http://target.ctf -tags default-login

# Tìm tech stack đang dùng
nuclei -u http://target.ctf -tags tech -s info
```

### CVE & Known Exploits

```bash
# Quét tất cả CVE
nuclei -u http://target.ctf -t cves/ -s critical,high

# Quét CVE cụ thể
nuclei -u http://target.ctf -t cves/2021/CVE-2021-44228.yaml   # Log4Shell
nuclei -u http://target.ctf -t cves/2022/CVE-2022-22965.yaml   # Spring4Shell
nuclei -u http://target.ctf -t cves/2021/CVE-2021-41773.yaml   # Apache Path Traversal
nuclei -u http://target.ctf -t cves/2021/CVE-2021-26855.yaml   # ProxyLogon (Exchange)
nuclei -u http://target.ctf -t cves/2020/CVE-2020-14882.yaml   # WebLogic RCE
```

### Fuzzing (CTF Web Challenges)

```bash
# Fuzz parameters
nuclei -u "http://target.ctf/page?id=FUZZ" -t fuzzing/

# SSRF detection
nuclei -u http://target.ctf -tags ssrf

# SSTI (Server-Side Template Injection)
nuclei -u http://target.ctf -tags ssti

# LFI (Local File Inclusion)
nuclei -u http://target.ctf -tags lfi

# XSS
nuclei -u http://target.ctf -tags xss

# SQL Injection
nuclei -u http://target.ctf -tags sqli

# Open Redirect
nuclei -u http://target.ctf -tags redirect
```

### OAST / Out-of-Band Testing

```bash
# Dùng Interactsh (built-in) để phát hiện blind vulnerabilities
nuclei -u http://target.ctf -tags oast

# Chỉ định interactsh server riêng
nuclei -u http://target.ctf -iserver https://your-interactsh.com -itoken <token>
```

### Workflow CTF phổ biến

```bash
# Bước 1: Xác định tech stack
nuclei -u http://target.ctf -tags tech,waf -s info -o tech.txt

# Bước 2: Quét broad (medium trở lên)
nuclei -u http://target.ctf -s medium,high,critical -o findings.txt

# Bước 3: Quét chuyên sâu CVE
nuclei -u http://target.ctf -t cves/ -s high,critical -json -o cves.json

# Bước 4: Tìm exposed items
nuclei -u http://target.ctf -tags exposure,panel,config -o exposed.txt
```

---

## 12. Lệnh Thường Dùng Trong Thi CERT / PenTest

### Quét Toàn Diện (Full Scan)

```bash
# Quét toàn bộ template với list mục tiêu
nuclei -l targets.txt -t nuclei-templates/ -o full-scan.txt -stats

# Quét với JSON output đầy đủ (bao gồm request/response)
nuclei -l targets.txt -json -irr -o full-scan.json

# Quét song song cao với rate limit
nuclei -l targets.txt -c 100 -bs 50 -rl 500 -o results.txt
```

### Phân loại theo Severity

```bash
# Chỉ tìm critical & high (báo cáo ưu tiên)
nuclei -l targets.txt -s critical,high -o critical-high.txt

# Quét medium để bổ sung báo cáo
nuclei -l targets.txt -s medium -o medium.txt
```

### SSL/TLS Audit

```bash
nuclei -l targets.txt -t ssl/ -o ssl-issues.txt
# Kiểm tra: expired certs, weak ciphers, misconfig TLS
```

### DNS Audit

```bash
nuclei -l domains.txt -t dns/ -o dns-issues.txt
# Kiểm tra: zone transfer, subdomain takeover, DNSSEC
```

### Network Services

```bash
# Kiểm tra exposed services
nuclei -l ips.txt -t network/ -o network-findings.txt

# Port-specific checks
nuclei -u tcp://target.com:22 -t network/ssh/
nuclei -u tcp://target.com:21 -t network/ftp/
```

### Subdomain Takeover

```bash
nuclei -l subdomains.txt -t takeovers/ -o takeovers.txt
```

### API Security

```bash
# Kiểm tra API endpoints
nuclei -l api-endpoints.txt -tags api -o api-issues.txt

# Swagger/OpenAPI exposed
nuclei -l targets.txt -tags swagger -o swagger.txt
```

### Cloud Misconfiguration

```bash
nuclei -l targets.txt -tags aws,gcp,azure -o cloud.txt
nuclei -l targets.txt -tags s3 -o s3-buckets.txt
```

### Quét qua Proxy (Burp Suite Integration)

```bash
# Forward traffic qua Burp để manual review
nuclei -l targets.txt -proxy http://127.0.0.1:8080 -t cves/ -s high,critical
```

### Tạo báo cáo Markdown

```bash
nuclei -l targets.txt -s medium,high,critical -markdown-export ./report/
```

### Tạo báo cáo SARIF (cho CI/CD)

```bash
nuclei -l targets.txt -sarif-export results.sarif
```

### Pipeline hoàn chỉnh (PenTest workflow)

```bash
# 1. Thu thập subdomains (kết hợp với subfinder/amass)
subfinder -d target.com -o subdomains.txt
httpx -l subdomains.txt -o live-hosts.txt

# 2. Quét với Nuclei
nuclei -l live-hosts.txt \
  -t cves/ \
  -t exposures/ \
  -t misconfiguration/ \
  -t vulnerabilities/ \
  -s medium,high,critical \
  -c 50 \
  -rl 200 \
  -json \
  -irr \
  -o pentest-results.json \
  -stats

# 3. Xuất báo cáo
nuclei -l live-hosts.txt -s high,critical -markdown-export ./final-report/
```

---

## 13. Tổng Hợp Flags Nhanh

```
TARGET:
  -u          URL mục tiêu
  -l          File danh sách URL
  -resume     Tiếp tục quét dở
  -sa         Quét tất cả IP của host

TEMPLATES:
  -t          Chỉ định template/thư mục
  -tl         Liệt kê templates
  -nt         Chỉ template mới
  -w          Workflow
  -validate   Kiểm tra template hợp lệ
  -eti        Loại trừ template

FILTERING:
  -s          Lọc theo severity (info/low/medium/high/critical)
  -es         Loại trừ severity
  -tags       Lọc theo tag
  -etags      Loại trừ tag
  -id         Lọc theo ID
  -author     Lọc theo tác giả

OUTPUT:
  -o          Lưu ra file
  -json       Xuất JSON
  -irr        Bao gồm request/response trong JSON
  -silent     Chỉ in kết quả
  -nc         Tắt màu
  -v          Verbose
  -debug      Debug mode (in full req/resp)
  -stats      Hiển thị thống kê
  -markdown-export  Xuất Markdown
  -sarif-export     Xuất SARIF

PERFORMANCE:
  -c          Concurrency (goroutines)
  -bs         Bulk size per host
  -rl         Rate limit (req/sec)
  -timeout    Timeout per request
  -retries    Số lần thử lại

NETWORK:
  -proxy      HTTP/SOCKS proxy
  -H          Custom HTTP header
  -V          Custom variable
  -r          DNS resolvers file

INTERACTSH (OOB):
  -iserver    Interactsh server URL
  -itoken     Interactsh token
```

---

## 📌 Tags Phổ Biến Cần Nhớ

|Tag|Mô tả|
|---|---|
|`cve`|Các CVE đã được đặt tên|
|`lfi`|Local File Inclusion|
|`rfi`|Remote File Inclusion|
|`ssrf`|Server-Side Request Forgery|
|`ssti`|Server-Side Template Injection|
|`xss`|Cross-Site Scripting|
|`sqli`|SQL Injection|
|`rce`|Remote Code Execution|
|`xxe`|XML External Entity|
|`idor`|Insecure Direct Object Reference|
|`redirect`|Open Redirect|
|`exposure`|Tệp/thư mục nhạy cảm bị lộ|
|`panel`|Admin/Login panels|
|`config`|File cấu hình bị lộ|
|`backup`|File backup bị lộ|
|`default-login`|Thông tin đăng nhập mặc định|
|`takeover`|Subdomain Takeover|
|`misconfiguration`|Cấu hình sai|
|`oast`|Out-of-Band Application Security Testing|
|`tech`|Phát hiện công nghệ|
|`waf`|WAF detection|
|`ssl`|SSL/TLS issues|
|`dns`|DNS misconfig|
|`api`|API security|
|`intrusive`|Các test có thể gây ảnh hưởng hệ thống|
|`fuzzing`|Fuzzing-based tests|

---

## ⚠️ Lưu Ý Quan Trọng

- **Tag `intrusive`** bị tắt mặc định vì có thể gây crash/hại hệ thống — chỉ dùng khi được phép.
- **Luôn xin phép** trước khi quét bất kỳ hệ thống nào ngoài môi trường lab/CTF.
- Trong CTF: ưu tiên quét `exposure`, `panel`, `default-login`, `cve` trước.
- Trong PenTest: kết hợp Nuclei với `subfinder + httpx + nmap` để có pipeline hoàn chỉnh.
- Dùng `-json -irr` khi cần lưu evidence đầy đủ cho báo cáo.

---

_Cheatsheet này được tổng hợp cho mục đích học tập, CTF và kiểm thử bảo mật hợp pháp._