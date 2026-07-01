---
title: "TryHackMe - Team"
author: Thien Nguyen
date: "2026-06-26"
subject: "CTF Writeup"
keywords: [THM, CTF, TryHackMe, Security, Offensive]
lang: "vi"

---


# Recon
## Nmap
- Trước tiên chúng ta sẽ dùng nmap để check port nào đang mở, version nào đang được sử dụng trên web application
```bash
sudo nmap -sC -sV -sS --reason -oA recon_nmap <target_ip>
# Nmap 7.95 scan initiated Fri Jun 26 00:27:08 2026 as: /usr/lib/nmap/nmap -sC -sV -sS --reason -oA recon_nmap <target_ip>
Nmap scan report for team.thm (10.49.148.80)
Host is up, received reset ttl 62 (0.091s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 62 vsftpd 3.0.5
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f0:7e:a4:0a:05:4d:6a:89:74:ea:5c:e6:d5:65:74:b6 (RSA)
|   256 00:04:e6:e7:16:ff:54:74:0f:b2:bd:78:f6:2d:d2:b4 (ECDSA)
|_  256 54:90:80:6c:28:e7:c2:d9:52:38:6a:80:3b:ac:fd:c0 (ED25519)
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Team
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jun 26 00:27:28 2026 -- 1 IP address (1 host up) scanned in 19.82 seconds

```

| port | service | version                                                       |
| ---- | ------- | ------------------------------------------------------------- |
| 22   | ssh     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0) |
| 80   | http    | Apache httpd 2.4.41 ((Ubuntu))                                |
| 21   | ftp     | vsftpd 3.0.5                                                  |


# Enumeration
- Sau khi đã recon trước các service đang sử dụng giờ sẽ tiếp hành enum
## fpt
- Trước tiên thì chúng ta thấy có 1 service ftp đang chạy trên server vì vậy chúng ta có thể thử đăng nhập ẩn anh để xem có thể truy cập vào không để tìm kiếm những gì có thể khai thác
```bash
└─$ ftp <target_ip>  
Connected to 10.49.148.80.
220 (vsFTPd 3.0.5)
Name (10.49.148.80:tsukuro): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp> exit
221 Goodbye.
```
- Yeah không thể vào được vì cần valid credential vậy chúng ta sẽ đổi sang bề mặt web có nhiều thứ để khai thác hơn 
## gobuster
- Sử dụng gobuster để kiểm tra tất cả dir và subdomain nào đang tồn tại 
-  dir
```bash
└─$ sudo gobuster dir -u http://<target_ip>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o gobuster_directionary.txt  
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.148.80/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 11366]
/server-status        (Status: 403) [Size: 277]
Progress: 4750 / 4750 (100.00%)
===============================================================
Finished
===============================================================

```
- dns
```bash
└─$ sudo gobuster dns --domain <target_ip> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster-dns.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     10.49.148.80
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 1006 / 4989 (20.16%)[ERROR] error on word fashion: lookup fashion.10.49.148.80.: i/o timeout
[ERROR] error on word syslog: lookup syslog.10.49.148.80.: i/o timeout
[ERROR] error on word t1: lookup t1.10.49.148.80.: i/o timeout
[ERROR] error on word teste: lookup teste.10.49.148.80.: i/o timeout
[ERROR] error on word welcome: lookup welcome.10.49.148.80.: i/o timeout
[ERROR] error on word neo: lookup neo.10.49.148.80.: i/o timeout
[ERROR] error on word sky: lookup sky.10.49.148.80.: i/o timeout
[ERROR] error on word exam: lookup exam.10.49.148.80.: i/o timeout
[ERROR] error on word webdisk.new: lookup webdisk.new.10.49.148.80.: i/o timeout
[ERROR] error on word img4: lookup img4.10.49.148.80.: i/o timeout
Progress: 3990 / 4989 (79.98%)[ERROR] error on word cupid: lookup cupid.10.49.148.80.: i/o timeout
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================

```
## Web Application Enummeration
![](../../05-Assets/Pasted%20image%2020260626122041.png)
- Như ảnh trên nó chúng ta có thể thấy đây chỉ là 1 trang apache homepage bình thường giờ thì kiểm tra source trang xem có manh mối gì không
![](../../05-Assets/Pasted%20image%2020260626122749.png)
- Oke giờ thì ta biết đây chỉ là trang default còn ip đang trỏ tới host 1 host khác là team.thm nó được gọi là **Virtual Hosting**(nơi 1 ip có thể host nhiều domain khác nhau). 
- Trước tiên phải chỉnh file host trên attack machine để truy cập được vào domain này
![](../../05-Assets/Pasted%20image%2020260626124340.png)
- Bây giờ đã có domain mới nên chúng sử dụng gobuster scan lại dir có thể tồn tại trên domain này
![](../../05-Assets/Pasted%20image%2020260701131229.png)
- Check file robots.txt thì ta thấy có tên 1 user dale
![](../../05-Assets/Pasted%20image%2020260701131310.png)




# Exploitation  



## User Flag


## Root Flag




# Conclusion
In the conclusion sections I like to write a little bit about how the box seemed to me overall, where I struggled, and what I learned.

# References
1. [https://ryankozak.com/how-i-do-my-ctf-writeups/](https://ryankozak.com/how-i-do-my-ctf-writeups/)
2. [https://github.com/Wandmalfarbe/pandoc-latex-template](https://github.com/Wandmalfarbe/pandoc-latex-template)
3. [https://hackthebox.eu](https://hackthebox.eu)
4. [https://forum.hackthebox.eu](https://forum.hackthebox.eu)