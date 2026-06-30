---
title: "TryHackMe - Recruit"
author: Thien Nguyen
date: "2026-06-30"
keywords: [THM, CTF, TryHackMe, Security, Offensive]
lang: "vi"

---


# Recon
## Nmap
```bash
└─$ sudo nmap -sC -sV -sS --reason -oA recon_nmap 10.48.161.169
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-30 04:33 EDT
Nmap scan report for 10.48.161.169
Host is up, received reset ttl 62 (0.10s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f2:df:ff:ed:ed:b7:b6:b0:ec:15:ab:bf:0e:ed:9a:91 (RSA)
|   256 a8:2a:a8:56:0e:5d:8d:a1:d1:37:ca:e8:5b:a0:57:e5 (ECDSA)
|_  256 a4:25:59:a8:94:fa:e4:96:63:a2:3e:a8:8a:f5:10:e4 (ED25519)
53/tcp open  domain  syn-ack ttl 62 ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Recruit
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.30 seconds

```
# Enumeration
## gobuster
### dir
```bash
└─$ sudo gobuster dir -u http://10.48.161.169/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o gobuster_directionary.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.161.169/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 315] [--> http://10.48.161.169/assets/]
/index.php            (Status: 200) [Size: 1417]
/javascript           (Status: 301) [Size: 319] [--> http://10.48.161.169/javascript/]
/mail                 (Status: 301) [Size: 313] [--> http://10.48.161.169/mail/]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://10.48.161.169/phpmyadmin/]
/server-status        (Status: 403) [Size: 278]
/sitemap.xml          (Status: 200) [Size: 1710]
Progress: 4750 / 4750 (100.00%)
===============================================================
Finished
===============================================================
```
- Có 4 nơi có thể tới là 
- http://10.48.161.169/assets/
- http://10.48.161.169/javascript/
- http://10.48.161.169/mail/
- http://10.48.161.169/phpmyadmin/
### DNS
```bash
└─$ sudo gobuster dns --domain 10.48.161.169 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster-dns.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     10.48.161.169
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================
```
=> Không có kết quả gì
## Web Application Enumeration
### index.html
![](../../05-Assets/Pasted%20image%2020260630155526.png)
- Có vẻ như đây là 1 trang đăng nhập thông thường đây có thể là attack sufface chúng ta có thể khai thác bây giờ thử kiểm tra source xem có gì có thể khai thác thêm gì không
![](../../05-Assets/Pasted%20image%2020260630155839.png)
=> Có 1 trang api.php
### api.php
- Đây chủ yếu là trang tổng hợp các câu hỏi thường gặp về API của web này nó là 1 trang nội bộ 
![](../../05-Assets/Pasted%20image%2020260630160742.png)
- Từ đây ta có thể biết những điều sau
+API Recruit được sử dụng nội bộ để lấy và xử lý hồ sơ ứng viên từ các nguồn bên ngoài trong quá trình tuyển dụng.
+Chúng ta có thể fetch cv của ứng viên thông qua endpoint sau: `/file.php?cv=<URL>`
+API còn hỗ trợ fetch CV từ các URL bên ngoài miễn nó hỗ trợ http và https
+Các request theo kiểu path traversal sẽ bị API block
### assets
![](../../05-Assets/Pasted%20image%2020260630161214.png)
- Chủ yếu là 1 trang tổng hợp các thư viện boostrap và css 
### javascript
![](../../05-Assets/Pasted%20image%2020260630161534.png)
- Ở đây forbidden đồng nghĩa chỉ có thể xem khi ssh được vào server
### mail
![](../../05-Assets/Pasted%20image%2020260630161624.png)
- 1 mail log trên server chứa nội dung tin nhắn của HR với bộ phận dev
![](../../05-Assets/Pasted%20image%2020260630162356.png)
- Các thông tin có thể có trong nội dung này là
+HR login username:hr
+Credentials của hr được lưu trong config.php
+Administrator credentials nằm trong database (không nằm trong file).
+Portal có: Login page, Candidate dashboard, API documentation.
### phpmyAdmin
- Đây là trang quản trị cơ sở dữ liệu mysql
![](../../05-Assets/Pasted%20image%2020260630163333.png)
- Hiện tại vẫn chưa thể đăng nhập được vì chưa có tài khoản nào cả 
### config.php
![](../../05-Assets/Pasted%20image%2020260630163450.png)
- Không trả về gì vậy buộc phải tìm cách exploit đọc nó
# Exploitation 
- Bây giờ quay lại endpoint `file.php?cv=` giờ chúng ta sẽ thử bypass SSRF path traversal để tìm cách đọc file config.php
![](../../05-Assets/Pasted%20image%2020260630172124.png)
- Đã có thông tin user hr sau khi đăng nhập vào chúng ta thấy dashboard và 1 thanh search dựa vào thông tin khi enum chúng ta biết được rằng credential của admin nằm trong database và database sử dụng MySQL
![](../../05-Assets/Pasted%20image%2020260630172541.png)
- Trước tiên chúng ta check xem có lỗi SQL Injection không thì nó đã trả lại lỗi như này đồng nghĩa lệnh SQL có thể như sau
```sql
SELECT * FROM candidates WHERE name LIKE '%input%'
```
![](../../05-Assets/Pasted%20image%2020260630172711.png)
- Bây giờ xác nhận xem những cột nào đang hiển thị trên trang
![](../../05-Assets/Pasted%20image%2020260630174254.png)
- Lấy tên database
![](../../05-Assets/Pasted%20image%2020260630174359.png)
- Lấy danh sách tên bảng
![](../../05-Assets/Pasted%20image%2020260630174441.png)
- Các cột có trang bảng user
![](../../05-Assets/Pasted%20image%2020260630174540.png)
- Lấy username và password có trong bảng
![](../../05-Assets/Pasted%20image%2020260630180835.png)
- Done vậy là leo lên được quyền admin
![](../../05-Assets/Pasted%20image%2020260630180929.png)
# Security Issues Identified

| Vulnerability          | Impact                       |
| ---------------------- | ---------------------------- |
| Information Disclosure | Leaked credentials location  |
| SSRF                   | Access to internal resources |
| Local File Inclusion   | Exposure of sensitive files  |
| Hardcoded Credentials  | Weak security practice       |
| SQL Injection          | Full database compromise     |
| Broken Access Control  | Privilege escalation         |
# Conclusion
- Ở CTF này chúng ta sẽ hiểu được nếu chỉ restrict việc không được đi theo kiểu path traversal là không đủ hacker luôn có nhiều cách để bypass nó
- SSRF là 1 pivot chứ không phải mục đích cuối cùng
- SQL Injection luôn nguy hiểm và luôn check nó đầu tiên khi khai thác các mục tiêu trong db
# References
1. https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/index.html?highlight=SSRF#ssrf-server-side-request-forgery
2. https://hacktricks.wiki/en/pentesting-web/sql-injection/index.html?highlight=sql%20in#sql-injection