# 🔐 Port Security Cheat Sheet

> Các mối đe dọa & điểm tấn công theo từng port phổ biến

---

## Tổng quan mức độ nguy hiểm

|Mức|Ký hiệu|Mô tả|
|---|---|---|
|Nghiêm trọng|🔴|Có thể dẫn đến RCE hoặc chiếm quyền hoàn toàn|
|Cao|🟠|Lộ thông tin nhạy cảm, leo thang đặc quyền|
|Trung bình|🟡|Khai thác có điều kiện, cần thêm bước|
|Thấp|🟢|Rủi ro thấp nếu cấu hình đúng|

---

## Port 21 — FTP

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|Brute Force|Thử username/password hàng loạt|
|Anonymous Login|Nhiều FTP server cho phép login ẩn danh mặc định|
|Cleartext Credentials|FTP không mã hóa → sniff được user/pass|
|Bounce Attack|Dùng FTP server làm proxy tấn công host khác|
|MITM|Chặn và thay đổi file trong quá trình transfer|
|Directory Traversal|Truy cập file ngoài thư mục cho phép (`../../../etc/passwd`)|

**Tools:** `hydra`, `nmap --script ftp-*`, `metasploit`

**Khuyến nghị:** Thay bằng SFTP (port 22) hoặc FTPS (port 990). Tắt anonymous login.

---

## Port 22 — SSH

**Mức độ:** 🟠 Cao (nếu hardened tốt: 🟢)

|Vector tấn công|Mô tả|
|---|---|
|Brute Force|Tools: hydra, medusa dò password|
|Username Enumeration|Timing attack lộ username hợp lệ (OpenSSH < 7.7)|
|CVE-2024-6387 (regreSSHion)|Unauthenticated RCE trên glibc Linux|
|Weak SSH Keys|RSA 1024-bit hoặc DSA key dễ crack|
|Authorized_keys Backdoor|Sau khi vào, attacker thêm key để persistence|
|SSH Tunneling Abuse|Bypass firewall, tạo tunnel vào mạng nội bộ|
|MITM (first connection)|Giả mạo server khi user chưa có known_hosts|

**Tools:** `ssh-audit`, `hydra`, `nmap --script ssh-*`

**Khuyến nghị:** Tắt password auth, chỉ dùng key, Fail2Ban, đổi port, tắt root login.

---

## Port 23 — Telnet

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|Cleartext Sniffing|Toàn bộ session không mã hóa, bao gồm password|
|Brute Force|Không có rate-limiting mặc định|
|MITM|Chặn và modify toàn bộ traffic|
|Credential Harvest|Sniff credentials từ mạng nội bộ|

**Tools:** `wireshark`, `tcpdump`, `hydra`

**Khuyến nghị:** **Tắt ngay lập tức.** Thay bằng SSH.

---

## Port 25 — SMTP

**Mức độ:** 🟠 Cao

|Vector tấn công|Mô tả|
|---|---|
|Open Relay|Server cho phép bất kỳ ai gửi mail → spam/phishing|
|User Enumeration|`VRFY`, `EXPN` commands lộ email hợp lệ|
|Email Spoofing|Giả mạo sender address nếu không có SPF/DKIM/DMARC|
|Bounce Attack|Dùng mail server làm relay tấn công|
|Brute Force Auth|Thử credentials SMTP AUTH|

**Tools:** `smtp-user-enum`, `swaks`, `nmap --script smtp-*`

**Khuyến nghị:** Tắt Open Relay, cấu hình SPF/DKIM/DMARC, dùng STARTTLS.

---

## Port 53 — DNS

**Mức độ:** 🟠 Cao

|Vector tấn công|Mô tả|
|---|---|
|DNS Zone Transfer|`AXFR` request lộ toàn bộ bản đồ mạng nội bộ|
|DNS Cache Poisoning|Chèn record giả vào cache resolver|
|DNS Amplification DDoS|Dùng DNS server làm amplifier DDoS|
|DNS Tunneling|Exfiltrate dữ liệu qua DNS queries (bypass firewall)|
|Subdomain Enumeration|Brute force tên subdomain|
|NXDOMAIN Attack|Flood queries để làm quá tải resolver|

**Tools:** `dig axfr`, `dnsenum`, `dnsrecon`, `iodine` (tunneling)

**Khuyến nghị:** Tắt zone transfer với external IP, rate-limiting, DNSSEC.

---

## Port 80 — HTTP

**Mức độ:** 🟠 Cao

|Vector tấn công|Mô tả|
|---|---|
|SQL Injection|Inject SQL qua form, URL parameter|
|XSS (Cross-Site Scripting)|Inject script độc hại vào trang web|
|Directory Traversal|Truy cập file hệ thống ngoài webroot|
|File Inclusion (LFI/RFI)|Include file nội bộ hoặc remote để RCE|
|IDOR|Thay đổi ID trong request để truy cập data của user khác|
|CSRF|Lừa browser nạn nhân thực hiện request trái phép|
|Clickjacking|Nhúng trang trong iframe để đánh lừa click|
|Web Shell Upload|Upload file PHP/JSP độc hại để RCE|
|Cleartext Credentials|Login form gửi password không mã hóa|

**Tools:** `burpsuite`, `nikto`, `sqlmap`, `gobuster`

**Khuyến nghị:** Chuyển sang HTTPS, WAF, input validation, Content Security Policy.

---

## Port 443 — HTTPS

**Mức độ:** 🟡 Trung bình

|Vector tấn công|Mô tả|
|---|---|
|SSL/TLS Misconfiguration|Dùng TLS 1.0/1.1, cipher yếu (RC4, DES)|
|BEAST, POODLE, HEARTBLEED|Các lỗ hổng SSL/TLS lịch sử còn tồn tại trên server cũ|
|Certificate Spoofing|Dùng cert giả (Let's Encrypt cho domain typosquatting)|
|Mixed Content|HTTP resources trong HTTPS page → downgrade|
|HTTP Request Smuggling|Khai thác sự khác biệt parse giữa proxy và server|
|Tất cả lỗ hổng web (port 80)|HTTPS chỉ mã hóa transport, không bảo vệ logic|

**Tools:** `testssl.sh`, `sslyze`, `burpsuite`

**Khuyến nghị:** TLS 1.2+, HSTS, strong cipher suites, OCSP stapling.

---

## Port 445 — SMB (Windows File Sharing)

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|EternalBlue (MS17-010)|RCE không cần xác thực, dùng bởi WannaCry/NotPetya|
|Pass-the-Hash|Dùng NTLM hash thay password để authenticate|
|NTLM Relay Attack|Relay authentication đến service khác|
|Null Session|Kết nối ẩn danh lấy thông tin share, user, group|
|Ransomware Propagation|SMB là vector lây lan chính của ransomware|
|Credential Brute Force|Thử user/pass trên SMB shares|

**Tools:** `nmap --script smb-*`, `metasploit`, `crackmapexec`, `responder`

**Khuyến nghị:** **Không expose SMB ra internet.** Vá MS17-010, tắt SMBv1, dùng firewall.

---

## Port 1433 — MSSQL

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|Brute Force|Thử SA account và các account mặc định|
|xp_cmdshell|Nếu bật → thực thi OS command trực tiếp từ SQL|
|SQL Injection|Truy cập DB không qua ứng dụng|
|Linked Server Abuse|Pivot sang server MSSQL khác trong mạng|
|Credential Dump|Đọc password hash từ sys tables|

**Tools:** `hydra`, `mssqlclient.py`, `metasploit`, `nmap --script ms-sql-*`

**Khuyến nghị:** Không expose ra internet, tắt `xp_cmdshell`, đổi port, strong password cho SA.

---

## Port 3306 — MySQL

**Mức độ:** 🟠 Cao

|Vector tấn công|Mô tả|
|---|---|
|Brute Force|Thử root và các user mặc định|
|Unauthorized Remote Access|Nhiều server bind `0.0.0.0` → ai cũng kết nối được|
|SQL Injection|Qua ứng dụng hoặc trực tiếp|
|UDF (User Defined Function)|Upload UDF độc hại để thực thi OS command|
|Information Schema Dump|Lấy tên DB, table, column|
|File Read/Write|`LOAD DATA INFILE`, `SELECT INTO OUTFILE`|

**Tools:** `hydra`, `sqlmap`, `mysqlcheck`, `nmap --script mysql-*`

**Khuyến nghị:** Bind localhost (`127.0.0.1`), xóa user anonymous, revoke FILE privilege.

---

## Port 3389 — RDP (Remote Desktop)

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|BlueKeep (CVE-2019-0708)|Unauthenticated RCE trước Windows 10|
|DejaBlue (CVE-2019-1181/1182)|Tương tự BlueKeep, ảnh hưởng Win 8/10|
|Brute Force|Bot quét liên tục 24/7, target Administrator account|
|Credential Stuffing|Dùng leak credentials từ các breach khác|
|Pass-the-Hash/Pass-the-Ticket|Dùng NTLM hash hoặc Kerberos ticket|
|Man-in-the-Middle|Chặn RDP session nếu dùng chứng chỉ tự ký|
|Ransomware Entry Point|RDP là vector số 1 để deliver ransomware|

**Tools:** `ncrack`, `hydra`, `crowbar`, `metasploit`

**Khuyến nghị:** **Không expose ra internet.** Dùng VPN, Network Level Authentication (NLA), đổi port, IP whitelist.

---

## Port 5900 — VNC

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|No Authentication|Nhiều VNC server không có password|
|Weak/No Encryption|VNC không mã hóa mặc định|
|Brute Force|Password ngắn dễ crack|
|Authentication Bypass|CVE cũ trong nhiều VNC implementations|
|Screenshot/Keylog|Chiếm màn hình và ghi keystroke|

**Tools:** `vncpwn`, `metasploit`, `hydra`

**Khuyến nghị:** Tunnel qua SSH, yêu cầu strong password, dùng TigerVNC/RealVNC với mã hóa.

---

## Port 6379 — Redis

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|Unauthenticated Access|Redis mặc định **không có password**|
|Write SSH Authorized Keys|Ghi SSH key vào `~/.ssh/authorized_keys` → SSH vào server|
|Cron Job Injection|Ghi crontab độc hại để RCE|
|Config File Overwrite|Ghi đè file cấu hình hệ thống|
|Data Dump|Đọc toàn bộ data trong cache (session, token, PII)|
|SSRF via Redis|Khai thác SSRF để tấn công Redis nội bộ|

**Tools:** `redis-cli`, `nmap --script redis-*`, `metasploit`

**Khuyến nghị:** **Bind localhost**, đặt requirepass, tắt CONFIG/SLAVEOF nếu không cần, rename nguy hiểm commands.

---

## Port 8080 / 8443 — HTTP Alternate / HTTPS Alternate

**Mức độ:** 🟠 Cao

|Vector tấn công|Mô tả|
|---|---|
|Admin Panel Exposure|Tomcat Manager, Jenkins, WebLogic thường chạy port này|
|Default Credentials|`admin:admin`, `tomcat:tomcat`, `admin:password`|
|Java Deserialization|WebLogic, JBoss, Jenkins có nhiều CVE deserialization|
|Path Traversal|Truy cập file ngoài webroot|
|Ghostcat (CVE-2020-1938)|AJP connector Tomcat cho phép đọc file tùy ý|

**Tools:** `burpsuite`, `nmap`, `metasploit`, `ysoserial`

**Khuyến nghị:** Đổi default credentials ngay lập tức, không expose admin panel ra internet.

---

## Port 27017 — MongoDB

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|No Authentication Default|MongoDB cũ mặc định không cần auth|
|Data Dump|Đọc toàn bộ database nếu không có auth|
|NoSQL Injection|Inject qua `$where`, `$gt`, `$ne` operators|
|Ransomware Target|Hàng nghìn MongoDB bị xóa data và đòi tiền chuộc|

**Tools:** `mongodump`, `nmap --script mongodb-*`, `nosqlmap`

**Khuyến nghị:** Bật authentication, bind localhost, dùng network ACL, MongoDB Atlas có auth mặc định.

---

## Port 9200 / 9300 — Elasticsearch

**Mức độ:** 🔴 Nghiêm trọng

|Vector tấn công|Mô tả|
|---|---|
|No Authentication|Phiên bản cũ không có auth → REST API mở hoàn toàn|
|Data Exfiltration|Dump toàn bộ index qua `/_cat/indices` và `/_search`|
|Remote Code Execution|CVE-2014-3120, CVE-2015-1427 (Groovy sandbox escape)|
|Kibana Attack Surface|Kibana trên port 5601 cũng thường không có auth|
|Ransomware|Hàng trăm nghìn Elasticsearch instance bị tấn công|

**Tools:** `curl`, `elasticdump`, `nmap`

**Khuyến nghị:** Bật X-Pack Security (auth + TLS), bind localhost hoặc dùng VPN.

---

## Bảng tổng hợp nhanh

|Port|Service|Nguy hiểm nhất|Mức độ|
|---|---|---|---|
|21|FTP|Cleartext creds, anonymous|🔴|
|22|SSH|Brute force, regreSSHion RCE|🟠|
|23|Telnet|Full cleartext sniff|🔴|
|25|SMTP|Open relay, spoofing|🟠|
|53|DNS|Zone transfer, amplification|🟠|
|80|HTTP|SQLi, XSS, LFI, web shell|🟠|
|443|HTTPS|TLS misconfig + web vulns|🟡|
|445|SMB|EternalBlue, ransomware|🔴|
|1433|MSSQL|xp_cmdshell RCE|🔴|
|3306|MySQL|Unauth access, UDF RCE|🟠|
|3389|RDP|BlueKeep, brute force, ransomware|🔴|
|5900|VNC|No auth, no encryption|🔴|
|6379|Redis|No auth, SSH key injection|🔴|
|8080|HTTP Alt|Admin panels, default creds|🟠|
|27017|MongoDB|No auth, data dump|🔴|
|9200|Elasticsearch|No auth, full data dump|🔴|

---

## Tools tổng hợp cho recon

```bash
# Scan port và service version
nmap -sV -sC -p- <target>

# Scan nhanh top ports
nmap -T4 -F <target>

# Scan với scripts NSE theo category
nmap --script vuln <target>
nmap --script auth <target>
nmap --script default <target>

# Kiểm tra SSL/TLS
testssl.sh <target>:443

# Web recon
nikto -h http://<target>
gobuster dir -u http://<target> -w /usr/share/wordlists/dirbuster/medium.txt
```

---

_Cheat sheet này chỉ dùng cho mục đích học tập, nghiên cứu bảo mật, và kiểm thử hệ thống mà bạn được phép kiểm tra._