## 1. Nikto là gì?

Nikto là một **web server scanner mã nguồn mở** viết bằng Perl, dùng để rà soát nhanh các vấn đề phổ biến trên web server: file/CGI nguy hiểm, phiên bản phần mềm lỗi thời, header cấu hình sai, file backup bị lộ, các thư mục mặc định nguy hiểm... Cơ sở dữ liệu của Nikto chứa hơn 6.700 mục kiểm tra (test) và hơn 1.200 phiên bản phần mềm đã biết lỗi.

Nikto **không phải** công cụ exploit — nó là công cụ **trinh sát (recon)**. Trong quy trình pentest hoặc CTF, Nikto thường được chạy ngay sau khi xác định được một cổng web đang mở (qua `nmap`), để nhanh chóng có "bản đồ" sơ bộ về những gì cần khai thác sâu hơn bằng tay hoặc bằng công cụ chuyên biệt khác.

## 2. Cài đặt

```bash
# Kali Linux / Parrot OS (đã cài sẵn)
nikto -Version

# Ubuntu/Debian
sudo apt update && sudo apt install nikto -y

# macOS (Homebrew)
brew install nikto

# Cài từ source (luôn có bản mới nhất)
git clone https://github.com/sullo/nikto.git
cd nikto/program
perl nikto.pl -h example.com

# Cập nhật database (rất quan trọng, nên chạy định kỳ)
nikto -update
```

## 3. Cú pháp cơ bản

```bash
nikto -h <target> [options]
```

Quét đơn giản nhất:

```bash
nikto -h http://10.10.10.10
```

Nếu không chỉ định scheme, Nikto coi đó là HTTP trên cổng 80. Với HTTPS:

```bash
nikto -h https://example.com
```

## 4. Bảng đầy đủ các tham số (options)

|Tham số|Mô tả|
|---|---|
|`-h, -host <target>`|Host/IP/URL mục tiêu. Có thể là file chứa danh sách host.|
|`-p, -port <port>`|Cổng cần quét (mặc định 80/443). Có thể liệt kê nhiều cổng: `-p 80,443,8080` hoặc dải `-p 1-1000`.|
|`-ssl`|Buộc dùng SSL/TLS cho kết nối.|
|`-o, -output <file>`|Lưu kết quả ra file.|
|`-F, -Format <type>`|Định dạng output: `csv`, `json`, `htm`, `sql`, `txt`, `xml`, `msf+` (Metasploit).|
|`-Tuning <số>`|Chỉ chạy nhóm test cụ thể (xem bảng Tuning bên dưới).|
|`-Plugins <name>`|Chỉ chạy plugin chỉ định, hoặc `-Plugins "@@ALL"` chạy tất cả.|
|`-id <user:pass>`|Cung cấp Basic Auth để quét trang yêu cầu đăng nhập.|
|`-Cgidirs <dir/all>`|Chỉ định danh sách thư mục CGI cần quét, hoặc `all`.|
|`-evasion <1-9>`|Kỹ thuật evasion theo kiểu LibWhisker (né IDS/IPS).|
|`-mutate <1-6>`|Đoán thêm file/đường dẫn bằng kỹ thuật biến đổi tên.|
|`-mutate-options <wordlist>`|Wordlist tùy chỉnh dùng cho `-mutate`.|
|`-T, -Tuning`|(alias cũ, một số bản dùng `-T`)|
|`-timeout <giây>`|Timeout cho mỗi request.|
|`-maxtime <thời gian>`|Giới hạn tổng thời gian quét, ví dụ `-maxtime 1h`, `-maxtime 30m`.|
|`-Pause <giây>`|Thời gian chờ giữa các test (giảm tải/né phát hiện).|
|`-D, -Display <V>`|Hiển thị thêm thông tin: `1` redirects, `2` cookies received, `3` requests sent, `4` full requests/responses, `E` lỗi.|
|`-C, -Cgidirs`|(xem trên)|
|`-vhost <hostname>`|Chỉ định Virtual Host header (Host header) khác với IP đích — rất hữu ích khi target chạy nhiều domain trên 1 IP.|
|`-useproxy`|Dùng proxy đã cấu hình trong `nikto.conf` (hoặc `-useproxy http://127.0.0.1:8080` để chạy qua Burp Suite).|
|`-nossl`|Không dùng SSL ngay cả khi cổng là 443.|
|`-no404`|Tắt việc tự động kiểm tra trang 404 (dùng khi server trả 404 bất thường gây false positive).|
|`-nointeractive`|Không cho phép tương tác trong lúc quét (hữu ích khi chạy script/automation).|
|`-nolookup`|Không resolve DNS.|
|`-Save <dir>`|Lưu các request/response thô vào thư mục để xem lại sau.|
|`-RSAcert <file>`|Sử dụng chứng chỉ client SSL khi target yêu cầu.|
|`-update`|Cập nhật plugin/database từ CIRT.net.|
|`-Version`|Hiển thị phiên bản Nikto và database.|
|`-Help`|Hiển thị trợ giúp đầy đủ.|
|`-ask <yes/no/auto>`|Có hỏi xác nhận khi phát hiện lỗi 404 bất thường không.|
|`-config <file>`|Dùng file config khác thay cho `nikto.conf` mặc định.|
|`-list-plugins`|Liệt kê tất cả plugin có sẵn.|
|`-IgnoreCode <codes>`|Bỏ qua các HTTP status code chỉ định (vd `-IgnoreCode 301,302`).|

## 5. Bảng `-Tuning` (rất hay dùng để thu hẹp phạm vi quét)

`-Tuning` giúp chỉ chạy nhóm kiểm tra mong muốn, giảm thời gian quét và giảm "noise". Có thể kết hợp nhiều số, hoặc dùng `x` để loại trừ.

|Mã|Loại kiểm tra|
|---|---|
|0|File Upload|
|1|Interesting File / Seen in logs|
|2|Misconfiguration / Default File|
|3|Information Disclosure|
|4|Injection (XSS/Script/HTML)|
|5|Remote File Retrieval - Inside Web Root|
|6|Denial of Service|
|7|Remote File Retrieval - Server Wide|
|8|Command Execution / Remote Shell|
|9|SQL Injection|
|a|Authentication Bypass|
|b|Software Identification|
|c|Remote Source Inclusion|
|d|WebService|
|e|Administrative Console|
|x|Reverse Tuning (loại trừ các mã đã chọn)|

Ví dụ thực tế:

```bash
# Chỉ quét Information Disclosure + Software ID + Admin console (quét nhanh khi cần thông tin nhanh)
nikto -h http://target -Tuning b,3,e

# Quét toàn bộ trừ Denial of Service (an toàn hơn cho production)
nikto -h http://target -Tuning x6
```

## 6. Các kịch bản sử dụng phổ biến

### 6.1. Quét cơ bản, nhanh

```bash
nikto -h http://target.com
```

### 6.2. Quét HTTPS với cổng tùy chỉnh

```bash
nikto -h https://target.com -p 8443 -ssl
```

### 6.3. Quét nhiều cổng/host cùng lúc

```bash
nikto -h target.com -p 80,443,8080,8443
```

```bash
# File hosts.txt chứa danh sách IP/domain, mỗi dòng một host
nikto -h hosts.txt
```

### 6.4. Xuất kết quả ra file để báo cáo

```bash
nikto -h http://target.com -o report.html -F htm
nikto -h http://target.com -o report.json -F json
nikto -h http://target.com -o report.txt -F txt
```

### 6.5. Quét qua Burp Suite / proxy (để xem từng request, debug, hoặc thêm vào Burp history)

```bash
nikto -h http://target.com -useproxy http://127.0.0.1:8080
```

### 6.6. Quét trang yêu cầu Basic Authentication

```bash
nikto -h http://target.com -id admin:password123
```

### 6.7. Giả lập Virtual Host (khi target dùng shared hosting / nhiều domain trên 1 IP)

Đây là kỹ thuật **rất phổ biến trong HackTheBox/CTF** khi máy chủ đích phục vụ nhiều subdomain trên cùng 1 IP nhưng phân biệt qua header `Host`:

```bash
nikto -h 10.10.10.10 -vhost blog.target.htb
```

### 6.8. Giới hạn thời gian quét (hữu ích khi target lớn hoặc đang luyện thi OSCP có giới hạn thời gian)

```bash
nikto -h http://target.com -maxtime 10m
```

### 6.9. Quét CGI directories

```bash
nikto -h http://target.com -Cgidirs all
```

### 6.10. Xuất sang định dạng Metasploit để liên kết module khai thác

```bash
nikto -h http://target.com -Format msf+
```

### 6.11. Bỏ qua các mã lỗi gây nhiễu (false positive do redirect/soft-404)

```bash
nikto -h http://target.com -IgnoreCode 301,302,404
```

### 6.12. Hiển thị chi tiết request/response để debug

```bash
nikto -h http://target.com -Display 4
```

## 7. Cách ippsec và các pentester chuyên nghiệp dùng Nikto trong thực chiến / HackTheBox

Qua các video walkthrough HackTheBox của ippsec, Nikto thường xuất hiện ở **bước recon ban đầu**, ngay sau `nmap`, theo một quy trình lặp lại nhiều lần:

**Bước 1 — Xác định cổng web bằng nmap**

```bash
nmap -sC -sV -p- target.htb -oN nmap-full.txt
```

**Bước 2 — Chạy Nikto song song với các công cụ recon khác (gobuster/feroxbuster, whatweb) để tiết kiệm thời gian**, vì Nikto có thể chạy khá lâu trên target có nhiều file:

```bash
nikto -h http://target.htb -o nikto.txt &
gobuster dir -u http://target.htb -w /usr/share/wordlists/dirb/common.txt -o gobuster.txt &
whatweb http://target.htb
```

ippsec thường nhấn mạnh: Nikto **giỏi nhất ở việc phát hiện**:

- Các file backup bị bỏ quên (`.bak`, `.old`, `.zip`, `~`)
- Header bảo mật thiếu (`X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`)
- Phiên bản server/CMS cũ có CVE công khai (Apache, nginx, PHP, WordPress, phpMyAdmin...)
- Các trang quản trị mặc định (`/admin`, `/manager/html`, `/phpinfo.php`)
- Thông tin tiết lộ qua các file như `robots.txt`, `.git`, `.svn`, `server-status`

**Bước 3 — Dùng `-vhost` khi nmap/whatweb gợi ý có virtual hosting**, đặc biệt với các máy HTB có domain dạng `*.htb` cần thêm vào `/etc/hosts` trước:

```bash
echo "10.10.10.10 target.htb" | sudo tee -a /etc/hosts
nikto -h target.htb
```

**Bước 4 — Đối chiếu kết quả Nikto với searchsploit/CVE** khi phát hiện phiên bản phần mềm cụ thể:

```bash
nikto -h http://target.htb | tee nikto-out.txt
# Khi thấy ví dụ "Apache/2.4.49" trong output:
searchsploit apache 2.4.49
```

**Một số lưu ý kinh nghiệm mà giới pentest uy tín (kể cả các bài viết trên trang chính thức CIRT.net, các writeup TJNull list, OSCP-prep blogs) hay nhắc tới:**

1. **Luôn `nikto -update` trước khi dùng** — database lỗi thời sẽ bỏ sót các test mới và phiên bản phần mềm mới.
2. **Nikto tạo rất nhiều request** (hàng nghìn) trong thời gian ngắn → dễ bị chặn bởi WAF/IPS, hoặc làm sập service yếu. Trên môi trường production cần cẩn trọng, có thể dùng `-Pause` để giảm tốc, hoặc loại trừ Tuning `6` (DoS).
3. **Kết quả Nikto luôn cần xác minh tay** — đây là công cụ có tỷ lệ false positive khá cao (đặc biệt là phần "Software Identification" dựa theo banner, và phần phát hiện file dựa theo HTTP status code). Không bao giờ đưa thẳng output Nikto vào báo cáo mà chưa kiểm chứng.
4. **Kết hợp với Burp Suite** qua `-useproxy` để vừa quét tự động vừa có log chi tiết phục vụ phân tích thủ công sau đó.
5. **Không thay thế được content discovery chuyên sâu** — Nikto không mạnh bằng `gobuster`/`feroxbuster`/`ffuf` khi cần brute-force thư mục với wordlist lớn tùy biến; nó chỉ kiểm tra một danh sách file/đường dẫn đã biết là "nguy hiểm" hoặc "thường gặp", nên thường chạy **song song**, không thay thế nhau.
6. **Trên các máy CTF retired (OSCP-like)**, Nikto hay được dùng để nhanh chóng phát hiện các lỗ hổng "low-hanging fruit" như phpMyAdmin lộ, file `.git` chưa xóa, hay banner phiên bản PHP/Apache cũ có exploit PoC sẵn trên Exploit-DB.

## 8. Quy trình quét khuyến nghị (tổng hợp best practice)

```bash
# 1. Update database
nikto -update

# 2. Quét toàn diện, lưu nhiều định dạng để vừa đọc nhanh (txt) vừa làm báo cáo (htm)
nikto -h http://target.com -o nikto-report -F htm,txt,json

# 3. Nếu nghi ngờ có virtual host
nikto -h target.com -vhost app.target.com

# 4. Nếu môi trường nhạy cảm / production, giảm tốc và tránh DoS test
nikto -h http://target.com -Tuning x6 -Pause 1

# 5. Đối chiếu version phần mềm tìm được với searchsploit / CVE database
searchsploit <tên-phần-mềm> <phiên-bản>
```

## 9. Đọc hiểu output của Nikto

Một dòng output Nikto điển hình:

```
+ Server: Apache/2.4.41 (Ubuntu)
+ /robots.txt: contains 2 entries which should be manually viewed.
+ The X-Content-Type-Options header is not set.
+ /admin/: This might be interesting.
+ /icons/: Directory indexing found.
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54).
```

Mỗi dòng `+` là một phát hiện độc lập. Các mục cần ưu tiên kiểm tra tay đầu tiên: directory indexing, các đường dẫn admin/quản trị, phiên bản phần mềm lỗi thời (tra CVE ngay), và các header bảo mật bị thiếu khi target là môi trường thực tế (không phải CTF).

## 10. Một số lệnh tổng hợp nhanh (quick reference)

```bash
nikto -h target.com                              # quét cơ bản
nikto -h target.com -p 80,443                    # nhiều cổng
nikto -h target.com -ssl                         # buộc SSL
nikto -h target.com -vhost sub.target.com         # virtual host
nikto -h target.com -id user:pass                 # Basic Auth
nikto -h target.com -o out.html -F htm            # xuất HTML
nikto -h target.com -Tuning b,3,e                 # chỉ chạy 1 số nhóm test
nikto -h target.com -useproxy http://127.0.0.1:8080  # qua Burp
nikto -h target.com -maxtime 5m                   # giới hạn thời gian
nikto -h target.com -Display 4                    # log chi tiết
nikto -h target.com -Format msf+                  # output cho Metasploit
nikto -update                                      # cập nhật database
nikto -Version                                      # xem phiên bản
nikto -list-plugins                                 # liệt kê plugin
```

## 11. Hạn chế của Nikto (cần biết để dùng đúng vai trò)

- Không quét được nội dung phía sau JavaScript rendering (SPA hiện đại như React/Vue cần công cụ khác).
- Dễ bị WAF chặn vì pattern request rất đặc trưng, dễ nhận diện.
- Không tự động khai thác (exploit) — chỉ phát hiện và báo cáo.
- Tỷ lệ false positive/negative tồn tại, đặc biệt với các server tùy biến trả về mã trạng thái khác chuẩn.
- Tốc độ quét toàn bộ database (6700+ test) trên target có nhiều thư mục có thể mất hàng chục phút đến vài giờ.

## 12. Tài liệu tham khảo

- Repo chính thức: https://github.com/sullo/nikto
- Wiki danh sách option đầy đủ: https://github.com/sullo/nikto/wiki/Annotated-Option-List
- CIRT.net (nguồn database gốc): https://cirt.net